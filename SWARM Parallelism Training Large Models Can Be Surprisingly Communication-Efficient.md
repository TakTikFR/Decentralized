---
Paper: https://arxiv.org/pdf/2301.11913
Year: "2023"
MOC: "[[Beyond A Single AI Cluster A Survey of Decentralized LLM Training]]"
---
---

## Introduction

Training large models is notoriously expensive due to the need for specialized HPC Clusters. So most researchers can't afford the experiment necessary for a proper evaluation of their ideas. This ultimately limits the scientific progress for many important research areas.

This paper aim to find a practical way of training large neural networks using **unreliable heterogeneous devices with slow interconnect**.

But to do so you have to find how to:
- Handle **peer failures**.
- Device **differences can generate bottleneck in the pipeline** by the weakest one.
- Handle **different network speeds**.
- **Dynamically rebalances nodes**.
- **Compress communication** for low bandwidth.
- Overcoming the problem of the **number of pipeline parallelism steps**.

## Background & Contributions 

In summary te paper make the following contributions:
- Formulate the **Square-Cube Law** of distributed training
	- a counter-intuitive observation that, for some methods, training larger models actually decrease the network overhead.

- Develop the **first decentralized algorithm capable of billion-scale training on heterogeneous unreliable devices with slow interconnect**.
	- Leverages randomized fault tolerant pipeline.
	- Dynamically rebalances nodes between pipeline stages.

- Compress the communication with **8-bit compression**.
	- Allow to train a billion-scale Transformer language model on preemptive servers with low-power GPUs and the network bandwidth of less than 200Mb/s while achieving high training throughput.

## Method

- **The Square-Cube Law**
	![[{5C0A1C82-8A30-4D0D-A047-EB6E535778B3}.png]]
	- The pipeline consists of $k$ stages, each represented by $n*n$ matrices, and 2 operations.
		- **Computation**, the compute time for naïve matrix multiplication is $O(n^3)$.
		- **Communication**, the communication phase requires at most $O(n^2)$.

	- So as we increase the model size, the computation time grows faster than the communication time.

	- In proportion to the compute and communication time, **the communication time will become less and less important proportionally**, as we can see in the right graph of the $Figure \space 1$.

	-  Then **more the model is important, more useful SWARM becomes**. Could be theoretically very useful for extremely large model as LLM.

- **SWARM Parallelism**
	![[{2E3C6E0B-2276-4F80-A90A-A55C528DFD6D}.png]]
	- Replace the rigid pipeline structure with **temporary pipelines**
		- Temporary pipelines are **built stochastically**.
		- Each peer send their output to any peer in the next pipeline stage.
			- The **next peer is determined on each iteration**.
			- If one peer is **faster than others**.
				- It'll **process inputs from multiple predecessors**
				- **Distribute its output to weaker peers** to **maximize utilisation**.
			- If one peer **disconnects**.
				- It predecessors reroute their request to its neighbors.
			- If one peer **joins**.
				- It'll **download up-to-date parameters and optimizer** from **remaining workers**.
		- This system consists of several consecutive swarms.
			- Peers within 