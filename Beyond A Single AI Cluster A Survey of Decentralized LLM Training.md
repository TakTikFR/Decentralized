---
Paper: https://arxiv.org/pdf/2503.11023v1
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

Due to bandwith limitations, it is essential to opitimize communication methods for decentralized model training. It can be done in 2 ways.

### 1. Reduce Communication Frequency

- Reduce the frequency of parameters synchronisation
- Larger local batches

### 2. Reduce Communication Intensity

- Dynamically forms group
- Compression
- Massive pipeline parallelism

## II. Heterogeneity Awareness

### 1. Network Heterogeneity

### 2. Device Heterogeneity

## III. Fault Tolerance