# Il modello non ci sta: le leve per usarlo lo stesso

Slide e fonti del community talk **"Il modello non ci sta: le leve per usarlo lo stesso"**, [mlopsday](https://www.mlopsday.it), Bologna, 29 ottobre 2026.

- **Slide:** [`slides/moe-streaming-mlopsday-2026.pdf`](slides/moe-streaming-mlopsday-2026.pdf)
- **Fonti di ogni numero:** [`FONTI.md`](FONTI.md)

## Un campo in rapida evoluzione

Tra luglio e ottobre 2026 sono usciti decine di motori che fanno girare un MoE più grande della memoria leggendo gli esperti da SSD. Cambiano linguaggio, hardware e modello; le idee sono le stesse, ed è su quelle che si concentra il talk.

| Progetto | Modello | Macchina | Velocità dichiarata |
|---|---|---|---|
| [Flash-MoE](https://github.com/danveloper/flash-moe) | Qwen3.5-397B-A17B, 209 GB a 4 bit | MacBook Pro M3 Max, 48 GB | ≈ 4,4 token/s |
| [Colibrì](https://github.com/alexandreofbh/colibri_GLM-5.2) | GLM-5.2, 744B, ≈ 370 GB a int4 | portatile da 25 GB di RAM; fino a M5 Max 128 GB | da 0,05 a 1,06 token/s |
| [llama.cpp, PR #25294](https://github.com/ggml-org/llama.cpp/pull/25294) (aperta) | GLM-5.2, ≈ 254 GB | GB10, 128 GB unificati | decode ≈ 2 token/s |
| [streamoe](https://github.com/transybao1393/streamoe) | Qwen3.6-35B-A3B, ≈ 19 GB a 4 bit | M3 Pro, tetto di 3–6 GB | 8–14 token/s |

I numeri sono dichiarati da autori e community. Gli esempi qui sotto usano soprattutto Flash-MoE, il caso meglio documentato.

## Il problema

Un MoE da circa 400 miliardi di parametri pesa circa 200 GB a 4 bit. Non sta nella memoria di nessuna singola macchina fuori dal datacenter:

| Hardware | Memoria | Banda | Costo indicativo |
|---|---|---|---|
| MacBook Pro M5 Max | 128 GB | 614 GB/s | ~6.700 $ |
| NVIDIA H100 | 80 GB | 3,35 TB/s | 25.000–40.000 $ |
| NVIDIA RTX 5090 | 32 GB | 1,79 TB/s | ~2.000 € |

Eppure i progetti qui sopra lo fanno funzionare su macchine da 25–128 GB.

## Perché si può: una parte fissa, esperti che cambiano

In un Mixture of Experts ogni layer ha due parti:

- **una parte fissa**: attention e router, usati a ogni token. È piccola e sta sempre in RAM (circa 5,5 GB in Qwen3.5-397B);
- **gli esperti**: centinaia di piccole reti feed-forward per layer. Il router ne sceglie pochi per ogni token, di solito tra 4 e 8; in molti modelli c'è anche un esperto condiviso sempre attivo.

Ogni layer ha i suoi esperti, e il layer successivo sceglie i propri solo quando riceve l'output del precedente. Per ogni token lavora una frazione dei parametri: 17 miliardi su 397 in Qwen3.5, circa 40 su 744 in GLM-5.2.

L'idea è una gerarchia di memoria: la parte fissa in RAM, tutti gli esperti su SSD, e in RAM salgono solo quelli scelti. Il costo per token è esperti attivi × dimensione di un esperto × numero di layer: circa 1,6 GB per Qwen3.5-397B (4 esperti da 6,75 MB per 60 layer), circa 11 GB per GLM-5.2 (8 esperti da circa 19 MB per 75 layer), prima della cache. Un modello denso della stessa taglia dovrebbe leggere tutti i pesi a ogni token.

## Le leve, ognuna con il suo prezzo

| Leva | Guadagni | Paghi |
|---|---|---|
| Quantizzazione | meno byte per token (Qwen3.5-397B: 209 GB a 4 bit, 120 GB a 2 bit) | a 2 bit si rompono JSON e tool calling |
| Cache | gli esperti popolari restano in RAM (~71% di hit) | RAM tolta al resto |
| Percorso di I/O | letture esplicite: `mmap` era 5 volte più lento | con memoria unificata SSD e GPU si contendono la banda: il prefetch costa il 73% alla GPU |

## Nascondere la latenza

Il router decide quali esperti servono solo quando arriva il suo turno, quindi la lettura da disco blocca il calcolo.

| Strategia | Guadagni | Paghi |
|---|---|---|
| On-demand | nessun errore | attesa a ogni layer |
| Speculativo | niente attesa, se indovini | se sbagli, due letture: in un caso reale −38% |
| Prerouting | lettura e calcolo in parallelo | il modello cambia un po', la qualità si recupera con una LoRA |

Con il prerouting un piccolo predittore per layer stima gli esperti del token successivo, e la sua previsione diventa il routing vero. Su un MoE da 35B passa da 3,9 a 20,4 token al secondo rispetto al caricamento on-demand.

## In una frase

Non serve più memoria. Serve sapere cosa tenerci. I motori cambiano ogni settimana: queste leve restano.

---

Francesco Montelli · [montelli.dev](https://montelli.dev) · francesco@montelli.dev
