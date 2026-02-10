---
Paper: https://arxiv.org/pdf/2503.11023v3
---
---

This paper is a survey about the state of the art in decentralized LLM training. It shows the advancement of decentralized LLM training from 2 points of view, organizational decentralization and community-driven decentralization, but mainly focus on the second point.

It exists 4 differents distributed machine learning paradigms:
- <u>Geo-Distributed Machine Learning</u>: Tackles service latency and regulatory needs by optimizing training processes accros datacenters.
- <u>Federated Learning</u>: Focuses more on privacy protection.
- <u>Decentralized Machine Learning</u>: It addresses communication bottlenecks inherent in parameter server architectures throught peer-to-peer networking, it offers a cost-effective and flexible strategy.
- <u>Efficient LLM Training</u>: Search to optimize 3 key aspect, memory utilization, communication overhead and computational efficiency.

---

The paper distinguish 3 ways to optimize decentralized LLM training.

## I. Communication Optimization

Due to **bandwidth limitations**, it is essential to **optimize communication** methods for decentralized model training. It can be done in 2 ways.

### 1. Reduce Communication Frequency

#### Data Parallelism

**Data Parallelism** splits the **input batch** across multiple devices. Each device holds a **complete model copy**, computes local gradients, then **averages gradients** via all-reduce for synchronized updates.

[[Gaia Geo-Distributed Machine Learning Approaching LAN Speeds|Gaia]]: 
	Gaia reduces communication frequency by designing **ASP (Approximate synchronous Parallel)** that achieves this with 3 techniques, the **significance filter**, the **selective barrier** and the **mirror clock**.

[[DeDLOC Distributed Deep Learning In Open Collaborations|DeDLOC]]:
	To compensate the relatively slow communication its **training with extremely large batches** and so **peers synchronise less frequently**.

[[DiLoCo Distributed Low-Communication Training of Language Models|DiLoCo]]:
	To reduce communications between devices DiLoCo uses **infrequent pseudo-gradient all-reduce**. It uses $H$, a value that represented the number of inner step before the synchronization with all-reduce ($H = 500$ in the paper).

#### Pipeline Parallelism

**Pipeline Parallelism** divides the **model layers** across devices (stages). **Micro-batches** flow through the pipeline like an assembly line, stages exchange **activations and gradients** between forward and backward passes.

[[SWARM Parallelism Training Large Models Can Be Surprisingly Communication-Efficient|SWARM]]:
	They formulate the **square-cube law**, a counter intuitive observation that for some methods training larger models **decrease the network overhead in proportion**. And only do the all-reduce when **enough gradients are accumulated**.

### 2. Reduce Communication Intensity

### Reduce Activations

[[SWARM Parallelism Training Large Models Can Be Surprisingly Communication-Efficient|SWARM]]:
	**SWARM reduces activation communication intensity** via **8-bit quantization** of activations exchanged between pipeline stages with **negligible quality loss**.

### Optimize Communication Topology

[[Ravnest Decentralized Asynchronous Training on Heterogeneous Devices|Ravnest]]:
	**Ravnest** uses **Multi-Ring Topology Coordination** for low-bandwidth networks.​
	It dynamically forms **multiple parallel All-Reduce rings**, balancing load on available links to maximize concurrent communication.

### Reduce gradients

[[OpenDiLoCo An Open-Source Framework for Globally Distributed Low-Communication Training|OpenDiLoCo]]:
	Used **FP16** and **PowerSGD** to compress gradients, thanks to that OpenDiLoCo shows that it can **halve the communication time** for the all-reduce operation.

## II. Heterogeneity Awareness

### 1. Network Heterogeneity

[[DeDLOC Distributed Deep Learning In Open Collaborations|DeDLOC]]:
	DeDLOC creates a **protocol** solving a linear program to **dynamically assign roles** (compute or aggregate) to peers based on measured bandwidth, **maximizing SGD throughput** (steps/sec) under heterogeneity.

### 2. Device Heterogeneity

[[SWARM Parallelism Training Large Models Can Be Surprisingly Communication-Efficient|SWARM]]:
	To handle heterogeneous devices SWARM uses different techniques, **Stochastic Wiring** that dynamically wire each input through each stage and pick devices in proportion to their training throughput. To optimize the usage of stochastic wiring they use **DHT (Distributed Hash Table)** to store connections and assigns data independently by using **IWRR (Interleaved Weighted Round-Robin) scheduler**.
[[Ravnest Decentralized Asynchronous Training on Heterogeneous Devices|Ravnest]]:
	Ravnest, developed a novel **asynchronous parallel training** approach that facilitate the distributed training of complex deep learning models over large datasets on clusters of heterogeneous consumer grade PCs.

## III. Fault Tolerance

[[DeDLOC Distributed Deep Learning In Open Collaborations|DeDLOC]]:
	DeDLOC uses **synchronous large‑batch data parallelism** so that any peer can **drop or join between steps** without corrupting the model state. Peers track global step counters via a **distributed hash table** and, if they fall behind or reconnect, they simply **download the latest parameters and optimizer state** from up‑to‑date peers before continuing. Gradient averaging is done in **small alternating groups** instead of everyone at once, so a failing node only breaks its local group, not the whole collaboration.

[[SWARM Parallelism Training Large Models Can Be Surprisingly Communication-Efficient|SWARM]]:
	SWARM is **pipeline‑parallel with redundancy inside each stage**, every stage is served by a **swarm of peers** that all hold the same layers. Trainers route micro‑batches to any live peer in the right stage, if one peer fails, requests are **re-routed to other peers in that stage** and training continues as long as at least one peer per stage is alive. Periodic **adaptive rebalancing** moves peers between stages to relieve bottlenecks when failures or speed differences appear.

[[OpenDiLoCo An Open-Source Framework for Globally Distributed Low-Communication Training|OpenDiLoCo]]:
	OpenDiLoCo inherits DiLoCo’s algorithm but runs it on **Hivemind’s peer‑to‑peer framework**, which explicitly targets fault tolerance. Coordination and metadata go through a **DHT**, and Hivemind’s state‑averager is designed so that **any worker can disappear at any time without stopping training**. Remaining peers continue local steps and synchronize with whoever is still online. There is **no master node**, so there is no single point of failure, robustness comes from decentralized coordination plus DiLoCo’s low communication frequency (workers only need to be alive at outer steps).

[[DiLoCo Distributed Low-Communication Training of Language Models|DiLoCo]]:
	DiLoCo itself is a **synchronous local‑SGD method**: workers do multiple local steps then average pseudo‑gradients with standard all‑reduce, and the paper assumes **reliable cluster‑style infrastructure**, not volatile peers. Fault tolerance is therefore mostly limited to what the underlying distributed framework provides.
