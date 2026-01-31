---
Paper: https://arxiv.org/pdf/2401.01728
Year: "2024"
MOC: "[[Beyond A Single AI Cluster A Survey of Decentralized LLM Training]]"
---
---

## Introduction

Modern deep learning models, growing larger and more complex, have demonstrated exceptional generalization and accuracy due to training on huge datasets. This trend is expected to continue. However, the increasing size of these models poses challenges in training, as traditional centralized methods are limited by memory constraints at such scales.

The Ravnest algorithm aims to facilitate distributed learning across a network of node without depending on a parameter server setup and each node having to operate on only a portion of the entire model.

The reasons for the need for Ravnest are as follows:
- Data parallelism needs **each node to store the full model**, impossible on weak/low‑RAM machines.
- Synchronous data parallelism **waits for the slowest worker**, so stragglers hurt efficiency.
- Parameter‑server setups become a **communication bottleneck** when many workers participate.
- Classic model parallelism is **sequential**: workers wait for inputs/gradients from neighbors that implies large **idle bubbles**
- Even pipeline parallelism still **requires frequent communication** and suffers when devices are **heterogeneous or over the internet**.
- Existing methods are not designed for **regular heterogeneous PCs** with limited bandwidth and reliability on the public internet.

## Background and Contributions

The main contributions of this paper are summarized as follows:
- Ravnest, a **novel asynchronous parallel training approach **.
	- Amalgamates the **best features of data and model parallelism**.
	- Facilitate the distributed training of complex deep learning models over large datasets on clusters of heterogeneous consumer grade PCs over the internet.

- A **Parallel Multi-Ring approach** which improves on single Ring All-Reduce has been utilized for parameter averaging.

- Their experiments show significant memory reductions and competitive convergence rates with conventional synchronous training, complementing the theoretical analysis.

## Method

- **Zero-Bubble Asynchronous Model Parallel Training**

	![[Pasted image 20260131160529.png]]

	- **Asynchronous Model Parallel is used within each cluster**
		- Eliminates the idle time bubble observed in traditional Model Parallel techniques.

	- Each cluster can be seen as a **computation graph**.
		- Each peer containing a sub model forms a node and communicates with a subset of peers in order to perform forward and backward passes.

	- **Forward**:
		- Receive activations from previous peer (One or more), in the forward buffer
		- Pop the buffer and do the forward pass on sub model
		- Send outputs to the next peer (One or more)

		- **No backward wait after the forward**.

	- **Backward**:
		- Receive gradients from next peer (One ore more, the order is reversed compere to forward), in the backward buffer.
		- Pop the buffer and do the backward pass on sub model
			- Compute gradients and send them to the previous peer (One or more).
			- Immediate local update of the parameters.

	- Thanks to the **asynchronous mechanism it produces zero idle bubbles**.

	- Each batch follows the same chain.
	- Queues are fill asynchronously.

- **Parallel Multi-Ring All-Reduce**

	![[Pasted image 20260131160604.png]]

	- The goal of this part is to **averaging parameters across all heterogeneous clusters efficiently**.
	- Number of rings = $max(nodes, clusters)$.
	- **Each ring correspond to a specific chunk of parameters**.

	- The **Figure 4**  show what happens during 1 round.
		- **Node size are proportional to the sub model size**.
		- **Big nodes handle multiple chunks**, so they're sending more arrows.
		- All 20 arrows ($5 \space rings * 4 \space clusters$) are **sent simultaneously**.

	-  During **1 round** all peers send chunks to the **next cluster peers** in **full parallel**.
	- A **Full All-Reduce** is **2×(C-1) rounds** for a complete averaging cycle, then all clusters have identical globally-averaged parameters.

- **Ravnest Algorithm**
	- The participating nodes are grouped together into **clusters based on their RAM and bandwidth**.
		- **No cluster is significantly more capable than the others**.
	- Each node is **assigned to a sub model that it's capable of training**.
	- Each **cluster can be viewed as a graph of interconnected nodes**.
		- Each individual **node is responsible for a portion of the model**.
	- All workers across all **clusters run parallelly in an asynchronous fashion**.
		- Zero-Bubble Asynchronous Model Parallelism is used within each cluster.
	- Updates are made as and when Gradients are received by a peer.
	- Model **parameters are averaged periodically** across clusters using the **reduction mechanism**.
	

## Results & Limitations

- **Results**
	- **Novel asynchronous parallel decentralized training paradigm**.
		- Leverages the strengths of both data and model parallelism techniques.
	- **Parallel Multi-Ring All-Reduce**.
		- Efficient parameter averaging across clusters and heterogeneous consumer grade PCs.
	- **Zero-bubble Asynchronous Model Parallelism**
		- Within each cluster further minimizes the idle time bubble on every node.
	- **Validate the convergence** and linear speedup characteristics.
	- **Memory reduction**, competitive convergence against synchronous baselines.

- **Limitations**
	- **No large LLMs** tested.
	- 