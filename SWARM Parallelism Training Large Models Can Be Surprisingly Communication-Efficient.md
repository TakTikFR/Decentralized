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
	- Replace the rigid pipeline structure with **temporary pipelines**.
		- Temporary pipelines are **built stochastically**.

		- Each peer send their output to any peer in the next pipeline stage.
			- The **next peer is determined on each iteration**.
			- If one peer is **faster than others**.
				- It'll **process inputs from multiple predecessors**.
				- **Distribute its output to weaker peers** to **maximize utilisation**.
			- If one peer **disconnects**.
				- It predecessors reroute their request to its neighbors.
			- If one peer **joins**.
				- It'll **download up-to-date parameters and optimizer** from **remaining workers**.

		- This system consists of several consecutive swarms.
			- Peers within one swarm serve the same pipeline stage.
				- **Same subset of layers** with the **same parameters**.
				- **Partitioned into evenly sized stages**.

		- **Forward pass**.
			- Peers **receive inputs from predecessors**.
			- Then **send activations to peers in the next stage**.

		- **Backward pass**.
			- Peers **receive gradients** for outputs.
			- **Compute gradients** for layer inputs and **accumulate gradients** for parameters.
			- Once enough gradients are accumulated.
				- Peers **form groups within their swarm**.
				- **Run All-Reduc**e to average gradients.
				- **Perform the optimizer step**.

	- Swarm can use **DPU (Delayed Parameter Updates)**.
		- **Improve hardware utilisation**.
			- Performing optimizer step in parallel with processing the next batch.

	- Each peer has **queues for incoming and outgoing request**.
		- **Maintain high GPU utilization** under latency
		- **Compensate for varying network speeds**

	- **Stochastic wiring**.
		- **Dynamically 'wire'** each input through each stage and **pick devices in proportion to their training throughput**.
		- Better utilize heterogeneous devices and recover from fault.

	- Utilisation of **DHT (Distributed Hash Table)**.
		- Trainers use **DHT** to list active peers per stage.
		- Usage of **Interleave Weighted Round-Robin** scheduler.
			- Each peer is associated with the** total processing time over all previous requests**.
			- Assigns microbatch independently to the **best-performing** (smallest total processing time) peer.
		- Trainer don't use GPUs, makes it possible to **run multiple trainers per peer**.

	- **Adaptive sward rebalancing**.
		- Allows for **automatic rebalancing within a stage**, need **cross-stage for max speed**.

		- **Every $T$ sec** Check **queue size** per stage.
		- Move Peers from empty to full swarm.
		- Get parameters and optimizer from new neighbors.

		- New peers automatically go to the best stage.
		- More peers to compensate hard stages.

	![[{A9EC768D-AB44-4696-86AC-63D98109C49A}.png]]

## Results & Limitations

- **Results**
	- Can train **very large models (Tested up to 13B)**.
	- Pipeline reaches **89% GPU utilization** on large models.
	- **Effective rebalancing algorithm**, maintains throughput despite **30-50% peers crashing/leaving**.

- **Limitations**
	- **Ineffective pipeline on small models**, Worse than offloading (communication dominates).
	- Needs many peers, **more than 1 peer per stage** for fault tolerance.
	- **High latency sensitive**, Works up to 100ms, degrades beyond.

## Q&A

- ==**In 'stochastic wiring', what they mean by 'stochastic' ?**==
	Trainers track **past processing times** per peer, then **route** micro-batches to **fastest ones** more often with **IWRR (Interleaved Weighted Round Robin)**.
- ==**Does peers train with the same batches ?**==
	Each peer train with a distinct micro batch (**1 peer = 1 distinct micro-batch**)
- ==**In the 'SWARM Parallelism' schema, what that means when a peer receives data from multiple predecessors ?**==
	The wires between **peers of different stages send only activations**.
	So when a peer receives from multiple predecessors, it receives more activations trained with different micro-batches.