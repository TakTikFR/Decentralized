---
Paper: https://www.usenix.org/system/files/conference/nsdi17/nsdi17-hsieh.pdf
tags:
---
---

# Introduction
2 - 4 phrases avec le contexte général (fédéré, decentralized, etc...)
Le problème precis qu'attaque le papier et pourquoi c'est important

Due to the expansion of LLM the need of large computing power to train those models increased, but datacenters have limited space storage, so the idea is to distribute this training among geographical distributed datacenters.

The goal of this paper is to develop a **geo-distributed ML system**, with **smart communication** mechanism and **generic and flexible** enough to run a wide range of ML algorithm.

To achieve these goals, such a system needs to address **two key challenges**:
- Efficiently use the limited and heterogeneous WAN bandwidth, because
	- ML algorithm requires constant and regular communication to exchange updates to keep global ML up to date.

- Design a general system that effectively handles communication for ML algorithms.
	- Communication patterns vary significantly across different ML algorithms.

# Background and Motivation
- Bullet points très concis :
    - Idée centrale du papier, en une phrase.
    - Contributions listées en langage à toi (pas recopiées).
- Optionnel : une mini phrase par contribution expliquant ce que ça change pour toi (lien à ton sujet).

The paper made 3 major contributions:
- This is the first work to propose a general geo-distributed ML system that:
	- Differentiates communication over LAN and WAN.
	- Is general and flexible to deploy a large range of ML algorithms.

- New efficient ML synchronisation model.
	- **ASP** (**A**ppoximate **S**ynchronous **P**arallel) optimize the WAN communication across datacenters.
	- Provides a theoretical guarantee on algorithm convergence.

- Provides significant performance improvements over two state-of-the-art distributed ML systems by sygnificantly reducing the communication overhead over WANs.

# Method
- Décrire uniquement ce qui est essentiel pour toi :
    - Architecture / algo / protocole expérimental, avec mots-clés techniques.
    - Hypothèses fortes et choix de design (topologie, compression, loss, dataset…)
- Si besoin, sous-sections :
    - 3.1 Modèle / Algo
    - 3.2 Protocole expérimenta


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
	- 

# Results and Limitations
- Résultats :
    - Où le papier brille (datasets, métriques, scénarios).
    - Ce qui est vraiment utile/réutilisable pour ton projet.​
- Limites (très important pour toi) :
    - Ce qu’ils ne testent pas, hypothèses irréalistes, aspects non adressés (scalabilité, hétérogénéité…).


# Q&A

- Difference between LAN and WAN ?
- What's BSP and SSP ?
- Why ASP is approximately correct ?