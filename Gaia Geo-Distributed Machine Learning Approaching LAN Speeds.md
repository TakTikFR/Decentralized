---
Paper: https://www.usenix.org/system/files/conference/nsdi17/nsdi17-hsieh.pdf
Year: "2017"
MOC: "[[Beyond A Single AI Cluster A Survey of Decentralized LLM Training]]"
---
---

# Introduction

Due to the expansion of LLM the need of large computing power to train those models increased, but datacenters have limited space storage, so the idea is to distribute this training among geographical distributed datacenters.

The goal of this paper is to develop a **geo-distributed ML system**, with **smart communication** mechanism and **generic and flexible** enough to run a wide range of ML algorithm.

To achieve these goals, such a system needs to address **two key challenges**:
- Efficiently use the limited and heterogeneous WAN bandwidth, because
	- ML algorithm requires constant and regular communication to exchange updates to keep global ML up to date.

- Design a general system that effectively handles communication for ML algorithms.
	- Communication patterns vary significantly across different ML algorithms.

# Background & Contributions

The paper made 3 major contributions:
- This is the first work to propose a general geo-distributed ML system that:
	- Differentiates communication over LAN and WAN.
	- Is general and flexible to deploy a large range of ML algorithms.

- New efficient ML synchronisation model.
	- **ASP** (**A**ppoximate **S**ynchronous **P**arallel) optimize the WAN communication across datacenters.
	- Provides a theoretical guarantee on algorithm convergence.

- Provides significant performance improvements over two state-of-the-art distributed ML systems by significantly reducing the communication overhead over WANs.

# Method

![Gaia system overview](gaia_system_overview.png)

In Gaia, each data center has some:
- **Worker Machines**
	- Each processes a **shard of the input data**.
	- Only **reads** and updates **the** **global model copy** in its data center.

- **Parameters Servers**
	- In each data center **collectively maintain a global model copy**.
	- Each handles a **shard of this global copy**.

Communication over LANs is effective and fast, Gaia uses two models to synchronize parameters within a data center, **BSP** (**B**ulk **S**ynchronous **P**arallel) or **SSP** (**S**tale **S**ynchronous 
**P**arallel). They can even use more aggressive communication schemes.

To reduce communication overhead over WANs they designed ASP, its goal is to ensure that the global model copy in each data center is approximately correct.

To achieve this, ASP is using 3 techniques:
- **The significance filter**
	- We know that more than 95% of the updates produce less than 1% change to the parameter value
	- It only sends parameters if their aggregated updates become significant enough.
	- ASP automatically reduces the significance threshold over time (if $v$ is the original threshold, then at the $t$ iteration the threshold would be $\dfrac{v}{\sqrt{t}}$)

With ASP for significant updates the WAN bandwidth might still be insufficient and in this case the updates can arrive too late, to handle this we use,
- **Selective barrier**
	- Main server sends just the list of important update indexes first (not values), telling other centers to pause reading those parameters.
	- Local workers stop reading affected data until the real updates arrive, keeping everyone in sync within set time limits.
	- Workers keep going on other tasks while waiting only for key updates, avoiding long stops.

With ASP selective barrier, WAN issues like fluctuating bandwidth or high latency can still delay significant updates. To fix this, we use 
- **Mirror Clock**:
	- Each parameter server shares its local "clock" (iteration count) with mirror servers in other data centers holding the same parameters.
	- If a fast server's clock gets ahead of the slowest mirror by a set limit (DS threshold), it pauses local workers from reading parameters until the slow one catches up.
	- Guarantees all workers stay in sync regardless of network problems, used only as last resort to ensure convergence (similar to SSP but more targeted).
# Results & Limitations

- **Results**:
	- Runs ML algorithms efficiently across WANs (1.8–53.5× faster than state-of-the-art).
	- Flexible sync (ASP) works over LANs and heterogeneous WANs.
	- Efficiently uses scarce WAN bandwidth by skipping insignificant updates (>95% of cases).
	- Guarantees ML algorithm convergence.

- **Limits**:
	- Assume relatively low and stable WAN latency, mirror clock as a last resort only, may block if extreme fluctuations occur.
	- No evaluations of high WAN latency or faults (focus bandwidth).
	- WAN costs remain high despite optimizations.
# Q&A

- ==**Difference between LAN and WAN ?**==
	**LAN (Local Area Network)**, responsible for the intra data center communication between worker machines and parameter servers.
	
	**WAN (Wide Area Network)**, its in charge for communication between data centers.
- ==**What's BSP and SSP ?**==
	**BSP (Bulk Synchronous Parallel)**: Strict sync where all workers compute, communicate, then wait at a global barrier before next iteration. No staleness, but slow with stragglers.

	**SSP (Stale Synchronous Parallel)**: Sync with bounded staleness (max s clocks apart), workers advance without waiting for everyone, but params may be stale (limit s). Faster than BSP, proven convergence.
- ==**Why ASP is approximately correct ?**==
	ASP sends only "significant" updates (threshold > ε), skipping small ones, approximates exact sync with less bandwidth. Proven to converge like exact async SGD under assumptions (bounded gradients, step size).
- ==**For ASP selective barrier, does it send updates in small batches if bandwidth is too low?**==
	Yes, indexes are sent first, values follow in background. If WAN saturated, values trickle in batches over time (bounded latency), workers pause only on affected params meanwhile.
- ==**What does "aggregated" mean in ASP**==
	Aggregated = cumulative local updates summed until threshold met (e.g., ||sum|| > ε). Once sent significant part, that component resets to zero; others continue aggregating independently per param.