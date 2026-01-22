---
Paper: https://arxiv.org/pdf/2106.10207
Year: "2021"
---
---

## Introduction 
**2–4 phrases courtes :** 
- Contexte général du domaine (WAN limits, geo-distributed ML, LLM scaling...). 
- **Problème précis** attaqué par le papier + pourquoi critique. 
- Objectif principal déclaré. 
- Optionnel : "Travaux liés clés" avec 2–3 `[[Papier-XYZ]]`.

## Background & Contributions 
**Bullet points très concis :** 
- **Sync existants** mentionnés : BSP/SSP/FedAvg (1 ligne chacun). 
- **Idée centrale** du papier, en une phrase. 
- **Contributions clés** listées en tes mots : 
	- Contrib 1 : [Architecture/système] → "Change X pour moi car Y". 
	- Contrib 2 : [Modèle sync innovant] → "Utile pour Z dans mon setup". 
	- Contrib 3 : [Perf/proofs] → "Benchmark vs state-of-art".

## Method 
**Focus techniques réutilisables :** 
- **Technique principale** : Algo/mécanisme avec formules clés (ε threshold, DS staleness).
- **Sous-techniques** : 2–3 détails implémentation (adaptive tuning, barriers...). 
- **Theoretical guarantees** : Convergence proof résumé (bounded gradients, step size). 
- **Datasets/évals** : Algos testés (MF, NN, ImageNet si vision).

## Results & Limitations 
**Résultats (où papier brille) :** 
- Speedups chiffrés (X× vs baselines nommées). 
- Bandwidth savings (% skippés, WAN usage). 
- Ce qui est **réutilisable** pour décentralisé images.
**Limites (très important) :** 
- Non testé (faults, high latency, heterogeneity). 
- Assomptions irréalistes (WAN stable). 
- Scalabilité non prouvée. 
- Overhead tuning/implém.

## Q&A 
**Format structuré Q:/R: → Réponses 1–2 lignes max**