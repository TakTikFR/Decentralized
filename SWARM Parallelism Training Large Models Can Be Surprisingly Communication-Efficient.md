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
- Handle **devices with different hardware and network speeds**.
- **Dynamically rebalances nodes**.
- **Compress communication** for low bandwidth.

## Background & Contributions 

In summary te paper make the following contributions:
- Formulate the **Square-Cube Law** of distributed training
	- a counterintuitive observation that, for some methods, training larger models actually decrease the network overhead.

- Develop the **first decentralized algorithm capable of billion-scale training on heterogeneous unreliable devices with slow interconnect**.
	- Leverages randomized fault tolerant pipeline
	- Dynamically rebalances nodes between pipeline stages.

- Compress the communication with **8-bit compression**.
	- Allow to train a billion-scale Transformer language model on preemptible servers with low-power GPUs and the network bandwidth of less than 200Mb/s while achieving high training throughput.

## Method