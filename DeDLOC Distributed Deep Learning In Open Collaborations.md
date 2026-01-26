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
- **Ensuring training consistency**.
	- **Synchronous data-parallel** training with **fixed hyper parameters**.
	- Training with **extremely large batches**.
		- Compensate the relatively **slow communication** by doing **less synchronisations**.
		- Peer **communicate less frequently**.
		- Natural way to **deal with heterogenous hardware**.
	- Each device accumulates gradients until the collaboration **reaches the target batch size**.
	- Devices exchange their gradients and **perform one optimizer step**.

-  **Adaptive averaging algorithm**
	Running efficient training on this kind of infrastructure requires a protocol that can *dynamically assign roles to every peer given their hardware and network capabilities*.
	- Dynamically solves a linear program to **maximize SGD throughput**.
		- by **assigning roles based on peer capabilities**.
			- compute performance, bandwidth and pairwise links.
	- Each peer aggregate a fraction of gradients proportionally to its capacity.
		- Fastest peers receive more and the slowest peers less.
	- Uses **Delayed Parameters Updates (DPU)** for compute and communication overlap.
	- Change global averaging with several consecutive iterations in alternating groups of size $m$.
		- Groups are defined to obtain the exact average in $\log_mn$ steps.
		- Apply All-Reduce by groups.
		- $m$ is bases on the number of peers and their failure rates.

- **System design**
	- DeDLOC ensures that all peers use up-to-date parameters.
		- By tracking the number of global steps of each peer.
		- If a peer is late, it'll downloads latest parameters and optimizer stats from up-to-date peers before resuming.
	- Requires **backbone peers** 
		- Stable connection and available at all times (At least one of them).
		- Responsible for welcoming new peers.
		- Performing auxiliary functions.
		- Can be hosted on unexpensive servers without GPU.
	- **Regular peers**
		- Depending on their hardware and network bandwidth, they can be assigned to:
			- Compute gradients.
			- Aggregate gradients computed by other peers.
			- Do both compute and aggregate gradients, according to the adaptive averaging algorithm.
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