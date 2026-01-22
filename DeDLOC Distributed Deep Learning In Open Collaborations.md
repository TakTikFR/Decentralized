---
Paper: https://arxiv.org/pdf/2106.10207
Year: "2021"
MOC: "[[Beyond A Single AI Cluster A Survey of Decentralized LLM Training]]"
---
---

## Introduction 

Using collaborative paradigm for machine learning is difficult due to **high latency**, **asymmetric bandwidth**, and **several challenges unique to volunteer computing**.

This paper designed strategies to allows many independent parties to combine their computational resources, and come up **DeDLOC (Distributed Deep Learning in Open Collaborations)**. Its based on a novel algorithm that adapts to the available hardware in order to maximize the training throughput and accommodate a large number of heterogeneous devices with uneven compute, bandwidth, reliability and networks capabilities.

But there is 3 major problems:
- A **big range of possible devices** that contribute to the collaborative experiments.
	- It can goes from GPU to even smartphone.

- **Different internet connection** with **limited bandwidth** and **low reliability**.

- It must handle **devices disconnections** from the training network.

## Background & Contributions 

The contributions can be summarized as follows:
- Propose a practical recipe for training in collaborative paradigm.

- Novel Distributed training algorithm to maximize the training performance

- Achieved competitive results for collaborative training under realistic conditions, with hundreds of data center GPUs.

## Method 

To overcome these problems, DeDLOC approaches the issue from three angles:
- **Ensuring training consistency**

-  **Adaptive averaging algorithm**

- **System design**
## Results & Limitations 
**Résultats (où papier brille) :** 
- Speedups chiffrés (X× vs baselines nommées). 
- Bandwidth savings (% skippés, WAN usage). 
- Ce qui est **réutilisable** pour décentralisé images.
**Limites (très important) :** 
- Non testé (faults, high latency, heterogeneity). 
- Assomptions irréalistes (WAN stable). 
- Scalabilité non prouvée. 
- Overhead tuning/implém.

## Q&A 
**Format structuré Q:/R: → Réponses 1–2 lignes max**