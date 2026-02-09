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
	

## III. Fault Tolerance