---
Paper: https://arxiv.org/pdf/2311.08105
Year: "2024"
MOC: "[[Beyond A Single AI Cluster A Survey of Decentralized LLM Training]]"
---
---

## Introduction

Large language models (LLM) have become a critical component in many applications of machine learning. However, standard approaches to training LLM **require a large number of tightly interconnected accelerators**, with devices exchanging gradients and other intermediate states at each optimization step.

In this work, they propose a **distributed optimization algorithm**, **Distributed Low-Communication (DiLoCo)**, that enables training of language models on islands of devices that are poorly connected. The approach is a variant of federated averaging, where the number of inner steps is large.

This paper try to address those problems:
- **Thousands of co-located devices** required in same data center.
- **Very high-bandwidth links** needed to minimize latency.
- **Complex software engineering** for gradient/parameter orchestration.
- **Higher failure risk** with more synchronous devices blocking training.
- **Poor heterogeneity handling** across varying device speeds or topologies.

## Contributions

Contributions can be listed as follow:
- **Dual-optimizer framework** with **Inner optimizer (AdamW)** and outer **optimizer (Nesterov)** on pseudo-gradients.
- **500x less communication** due to the infrequent pseudo-gradient all-reduce.
- It can scale to workers that are poorly connected.
- **DiLoCo is a robust and effective way to distribute training**.

## Method

![[Pasted image 20260207092311.png]]

The DiLoCo algorithm works as follows:
- Start from an i**nitial model with parameters** **$\theta^{(0)}$
	- Can be randomly initialized or using a pretrained model.
- We have $k$ **workers** and **shards of data**, one for each worker.
- There are **two optimization processes**.
	- **Outer optimization**.
		- Consist of $T$ outer steps.
		- At each outer step $t$, outer gradients from each worker are **gathered**, **averaged** and sent to an outer optimizer.
		- Outer optimizer **Updates the shared copy** of the parameters and **re-dispached** to each worker

		- The DiLoCo **outer optimizer** is **SGD** with **Nesterov momentum**.
	- **Inner optimization**.
		- Consist of H inner steps
		- At each step $h$, each worker **performs independently and in parallel its own inner optimization**.
		- Worker samples data from its own shard.
		- Updates its own **local copy of parameters**.

		- The DiLoCo **inner optimizer** is **AdamW**
			- It's the most widely used optimizer for transformer language model.

- Once all workers have completed their inner optimization step.
	- The delta (pseudo gradients) is computed on each worker.
		- Pseudo gradients : $\Delta = \theta^{(t - 1)} - \theta^{(t)}$ 
			- With $\theta^{(t - 1)} =$ Locally updated weight 
			- And  $\theta^{(t)} =$ Original weight
			- Compute manually and **store in FP32**, can be **store and all-reduced in FP16** without performance hit.
	- Averaged all the worker's delta in the outer gradient and update the shared copy with it. 
		- $\theta^{(t)} \leftarrow OuterOpt(\theta^{(t-1)}, \Delta^{(t)})$
		- Now **used as starting point** for the next round of inner optimizer

![[Pasted image 20260207095917.png]]

## Results & Limitations

- **Results**:
	- **Great robustness to the data distribution** of each worker.
	- **500 times less communication**, communication across workers is minimal.
	- Scale to workers that are **poorly connected**.
	- Trained on **400M parameters** transformer language model, across up to **64 replicas** and up to **100 times less communication**

- **Limits**:
	- Only considered **language modeling** with only **transformer architecture **.
	- Assumes that **all workers are homogeneous**.
	- Below **8 workers performance declines**.
## Q&A

- ==**Who compute the all-reduce and dispatch them ?**==
	**All workers perform the all-reduce collectively** using `torch.distributed.all_reduce(op=ReduceOp.AVG)`. No master node,the communication backend (NCCL or Gloo) uses **ring/tree algorithms** to compute the average and returns it directly to **every worker's buffer**.