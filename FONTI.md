# Fonti

Ogni numero del talk, con la fonte da cui viene. Verificate il 29 ottobre 2026.

## Il caso reale: Qwen3.5-397B-A17B su un MacBook da 48 GB

- [Flash-MoE](https://github.com/danveloper/flash-moe): Qwen3.5-397B-A17B su MacBook Pro M3 Max da 48 GB a 4,4 token/s; 512 esperti per layer, 4 attivi più uno condiviso; 60 layer; 209 GB a 4 bit e 120 GB a 2 bit; parte fissa di circa 5,5 GB; ~71% di hit della page cache; `mmap` 5 volte più lento di `pread`; prefetch con −73% sulla GPU per la contesa della memoria unificata; routing speculativo −38%; a 2 bit si rompono JSON e tool calling.
- [Flash-MoE, articolo tecnico](https://lilting.ch/en/articles/flash-moe-qwen35-397b-metal-inference): circa 6,75 MB per esperto a 4 bit, lettura dei 4 esperti per layer.

## Streaming degli esperti e prerouting

- [Edge0](https://www.alphaxiv.org/abs/2609.18063): prerouter per layer usato come routing, recupero della qualità con una LoRA; su un MoE da 35B da 3,9 a 20,4 token/s rispetto allo streaming on-demand.
- [FlashMoE](https://arxiv.org/abs/2601.17063): esperti su SSD e politiche di cache apprese.
- [Apple, LLM in a Flash](https://arxiv.org/abs/2312.11514): inferenza di LLM con memoria limitata leggendo i pesi dalla flash.

## Mixture of Experts

- [Hugging Face, Mixture of Experts Explained](https://huggingface.co/blog/moe): il punto di partenza.
- [Shazeer et al., Sparsely-Gated Mixture-of-Experts (2017)](https://arxiv.org/abs/1701.06538): router e top-k.
- [Switch Transformers](https://arxiv.org/abs/2101.03961): routing e loss di bilanciamento.
- [ST-MoE](https://arxiv.org/abs/2202.08906): cosa imparano gli esperti.
- [Mixtral of Experts](https://arxiv.org/abs/2401.04088): un MoE open molto noto.
- [Sparse Upcycling](https://arxiv.org/abs/2212.05055): MoE nati da un modello denso.

## Hardware e prezzi

- [NVIDIA H100](https://www.nvidia.com/en-us/data-center/h100/): 80 GB, 3,35 TB/s, NVLink 900 GB/s.
- [IntuitionLabs, prezzi delle GPU da datacenter 2026](https://intuitionlabs.ai/articles/data-center-gpu-pricing-2026): H100 nuova tra 25.000 e 40.000 $.
- [Apple, specifiche MacBook Pro 16" M5 Pro / M5 Max](https://support.apple.com/126319): fino a 128 GB di memoria unificata con M5 Max.
- [PetaPixel, M5 Pro e M5 Max](https://petapixel.com/2026/03/03/apple-debuts-m5-pro-and-m5-max-chips-for-demanding-pro-workflows/): 614 GB/s di banda per M5 Max.
- [ComputingForGeeks, miglior Mac per LLM locali](https://computingforgeeks.com/best-mac-for-local-llm/): MacBook Pro 14" M5 Max, 128 GB, 2 TB a 6.699 $ (prezzi del 27 ottobre 2026).
- [TechPowerUp, RTX 5090 Founders Edition](https://techpowerup.com/review/nvidia-geforce-rtx-5090-founders-edition): 32 GB GDDR7, 1,79 TB/s.
- [PCGH, RTX 5090 Founders Edition in Europa](https://extreme.pcgameshardware.de/threads/669421): prezzo di riferimento intorno ai 2.000 €.

## Note

- I pesi a 16 e 8 bit (≈ 800 e ≈ 400 GB) sono calcolati: parametri × bit ÷ 8.
- "Quasi identica" a 8 bit e "lieve perdita" a 4 bit descrivono il comportamento tipico della quantizzazione, non sono misure su questo modello.
- I prezzi sono indicativi e cambiano nel tempo.
