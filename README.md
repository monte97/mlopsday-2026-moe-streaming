# Streaming degli esperti: modelli MoE oltre la RAM

Slide e fonti del community talk **"Streaming degli esperti: modelli MoE oltre la RAM"**, [mlopsday](https://www.mlopsday.it), Bologna, 29 ottobre 2026.

- **Slide:** [`slides/moe-streaming-mlopsday-2026.pdf`](slides/moe-streaming-mlopsday-2026.pdf)
- **Fonti di ogni numero:** [`FONTI.md`](FONTI.md)

## Il problema

Qwen3.5-397B-A17B pesa 209 GB a 4 bit. Non sta nella memoria di nessuna singola macchina fuori dal datacenter:

| Hardware | Memoria | Banda | Costo indicativo |
|---|---|---|---|
| MacBook Pro M5 Max | 128 GB | 614 GB/s | ~6.700 $ |
| NVIDIA H100 | 80 GB | 3,35 TB/s | 25.000–40.000 $ |
| NVIDIA RTX 5090 | 32 GB | 1,79 TB/s | ~2.000 € |

Eppure gira su un MacBook Pro M3 Max con 48 GB, a circa 4,4 token al secondo.

## Perché si può: una parte fissa, esperti che cambiano

In un Mixture of Experts ogni layer ha due parti:

- **una parte fissa**: attention e router, usati a ogni token. È piccola e sta sempre in RAM (circa 5,5 GB in questo modello);
- **gli esperti**: 512 piccole reti feed-forward per layer. Il router ne sceglie 4 per ogni token, più un esperto condiviso sempre attivo.

Ogni layer ha i suoi esperti, e il layer successivo sceglie i propri solo quando riceve l'output del precedente. Per ogni token lavorano circa 17 miliardi di parametri su 397.

L'idea è una gerarchia di memoria: la parte fissa in RAM, tutti gli esperti su SSD, e in RAM salgono solo quelli scelti. Il costo è di circa 4 esperti da 6,75 MB per ciascuno dei 60 layer: circa 1,6 GB letti per token, prima della cache. Un modello denso della stessa taglia dovrebbe leggere tutti i pesi a ogni token.

## Le leve, ognuna con il suo prezzo

| Leva | Guadagni | Paghi |
|---|---|---|
| Quantizzazione | meno byte per token (209 GB a 4 bit, 120 GB a 2 bit) | a 2 bit si rompono JSON e tool calling |
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

Non serve più memoria. Serve sapere cosa tenerci.

---

Francesco Montelli · [montelli.dev](https://montelli.dev) · francesco@montelli.dev
