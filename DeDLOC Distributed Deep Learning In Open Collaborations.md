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

	- Each peer aggregate a fraction of gradients **proportionally to its capacity**.
		- Fastest peers receive more and the slowest peers less.

	- Uses **Delayed Parameters Updates (DPU)** for compute and communication overlap.

	- Change global averaging with several consecutive iterations in alternating groups of size $m$
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
		- Can be hosted on inexpensive servers without GPU.

	- **Regular peers**
		- Depending on their hardware and network bandwidth, they can be assigned to:
			- Compute gradients.
			- Aggregate gradients computed by other peers.
			- Do both compute and aggregate gradients, according to the adaptive averaging algorithm.

	- Allow participants to **download the data progressively during training**.
		- Once the first several exemples are obtained, it can begin the training
		- Loads shards in a different random order and shuffle the data.
			- Ensure training exemples are independent and identically distributed.
## Results & Limitations 
**Results**:
DeDLOC demonstrates effectiveness in vision (SwAV / ImageNet), controlled NLP (ALBERT / WikiText), and real-world applications (sahajBERT Bengali with 40 volunteers).

- Enables large scale distributed training regardless of hardware and network.
- Maintains its efficiency in a broad range of conditions.
- Faster than traditional methods on mixed infrastructure
- Scalable up to ~40 simultaneous active peers

**Limits**: 
- Need to gather a community of volunteers
- Need backbone server
- 2-3× less efficient FLOPs than conventional clusters
- Medium models only (not giant LLMs)

## Q&A 

- ==**How do they "naturally" handle heterogeneous speed of compute hardware ?**==
	They set a **global batch size target**, so each **participant contributes proportionally** to this target based on their capacity.
- ==**During communication, do peers train on all parameters but send only specific chunks ?**==
	Each peer is assigned to accumulate gradients over a specific fraction of model parameters.
	So everyone **compute each gradients on its own**, and then **cuts them into chunks** (As many chunks as peers assigned to accumulate the gradients), and finally **send the specific chunk to the associated peer**.
- ==**To handle node failures, what they mean by "Alternating groups" ?**==
	Replace global All-Reduce by hierarchical All-Reduce to handle node failures.
	At each steps create a new group of peers (m = 4-8, depends of failure rates), and apply the All-Reduce over this group of peers.
	Change the group at each steps by ensuring that no one falls back into the same group twice.
- ==**What are auxiliary functions ?**==
	Support functions for backbone peers: welcoming new collaborators (DHT handshake), storing checkpoints, tracking learning curves.
- ==**What's learning curves & how they tracked them ?**==
	Learning curves = loss / accuracy graphs vs time / steps. Tracked via backbone peers only.
- ==**How they store checkpoints ?**==
	All model parameters and optimizer states are stored on backbone peers.
- ==**How do you choose which device compute gradients, aggregate them or do both ?**==
	Role assignement via LP :$$max \space min (\frac{∑_{i=1}^n s_i c_i}{B},\frac{min_i ∑_j g_{ji}}{P}) = max \space min (F_{compute},F_{agg})$$
	- Both High $F_\text{compute}$ and $F_\text{agg}$
		- **Good compute** and **good bandwidth** 
		- **Compute** and **aggregate** 
	- Only $F_\text{agg}$ is high
		- **Good bandwidth**
		- **Aggregate gradients only** 
	- Only $F_\text{compute}$ is high
		- **Good compute**
		- **Compute gradients only**
- ==**How DPU works ?**==
	The Problem is that the sequential compute involves a loss of time during communication.
	So **DPU overlap with exactly 1 round staleness**.
	- Compute new gradients.
	- **In parallel**, send old gradients and receive new aggregated parameters.
	- Apply update and repeat.