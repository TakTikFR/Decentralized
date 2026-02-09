---
Paper: https://arxiv.org/pdf/2407.07852
Year: "2024"
MOC: "[[Beyond A Single AI Cluster A Survey of Decentralized LLM Training]]"
---
---

## Introduction

OpenDiLoCo is an open-source implementation and replication of the Distributed Low Communication ([[DiLoCo Distributed Low-Communication Training of Language Models|DiLoCo]]) training method for large language models. They provide a reproductible implementation of the DiLoCo experiments.

This paper also conduct an ablation study of DiLoCo, focusing on:
- The algorithm's scalability in the number of workers.
- Scalability in compute efficiency.
- Gradients compression with FP16.

## Contributions

The contributions of their work are as follows:
- **Reproduction and scaling of DiLoCo experiments**.
	- Replicate for a LLM.
	- Successfully extend the DiLoCo experiments to the **billion parameter model scale**.
- **Open source implementation**.
	- Enables single DiLoCo workers to **scale to hundreds of machines**.
- **Global decentralized training**.
	- Executed across **2 continents and 3 countries**.
	- Achieves **90-95% compute utilization**.
- **Analytical insights and ablations**.
	- Gradients can be **effectively all-reduced using FP16** without any performance degradation.

## Method

- Leverages two distinct optimization processes.
	- Inner optimizer with **AdamW**.
		- Perform local updates on individual workers.
	- Outer optimizer applies **SGD** and **Nesterov momentum** on all-reduced pseudo-gradients.
		- **Synchronizes the workers** using pseudo-gradients.
			- Pseudo gradients : $\Delta = \theta(t + h) - \theta(t)$ 
				- With $\theta(t + h) =$ Locally updated weight 
				- And  $\theta(t) =$ Original weight
				- Compute manually and **store in FP32**, can be **store and all-reduced in FP16** without performance hit.
		- Significantly **reduces the frequency of communication**, **up to 500 times**.
			- Lowering the bandwidth requirements.
	- In mixed precision training
		- With **FP16**, a **gradient scaler** is used to improve the dynamic range of the gradients.
			- Avoiding underflow and overflow.
			- The gradient scaler should be called during the inner optimization step.
				- Because pseudo-gradients are calculated manually in FP32.

- The implementation with **Hivemind**.
	- **Hivemind** is a **real-world decentralized** training setup
	- The **amount of available computing resources is not constant**, as new devices and clusters regularly join or leave the system.
	- Through Hivemind's fault-tolerant training, **a device could become unavailable at anytime without stopping the training process**.
	- There is **no master node**, all **communication is done in  a peer-to-peer fashion**.

## Results & Limitations

- **Results**
	- Scale the DiLoCo method to **3 times parameters size (400M -> 1.1B)**
	- Demonstrate its **application in real world** decentralized training setting.
	- Trained a LLM across **2 continents and 3 countries**.
	- Achieved **90-95% compute utilization**.
	- **Matches DDP baseline perplexity**.
	- **FP16 all-reduce is effective** with DiLoCo and **halve the communication time** for the all-reduce operation.

- **Limits**
	- Need to **scale DiLoCo to more than 8 workers**.
	- Have to **develop more compute-efficient methods** for decentralized learning.
	- Reduce the compute idle time
		- By performing the **weight averaging communication asynchronously**.
	- Scaling **beyond 8 workers** shows diminishing returns early in training.
	- **OpenDiLoCo converges slower early** due to **asynchronous noise**, with gains appearing later.

## Q&A

- ==**What's a gradient scaler ?**==
	**Gradient scaler** multiplies the loss by a large number (**Power of 2**) before backward pass so tiny FP16 gradients don't vanish to zero. Framework unscales them back to normal values before the optimizer step, **adjusting the multiplier dynamically** if overflow occurs.