---
title: "Quantum vs Classical IDS"
date: 2025-02-10
draft: true
summary: "Benchmarking a 4-qubit VQC against XGBoost on NetFlow data. Where quantum helps, where it doesn't."
tags:
  - "Quantum ML"
  - "Cybersecurity"
  - "PennyLane"
---

For Eigencurve, we built a hybrid quantum-classical intrusion detection system and benchmarked it against purely classical approaches. The question was simple: does a parameterized quantum circuit offer any advantage on non-linear decision boundaries in high-dimensional network traffic data?

## Setup

- **Quantum model**: 4-qubit variational quantum circuit using PennyLane with angle embedding and strongly entangling layers
- **Classical baselines**: XGBoost and LightGBM
- **Dataset**: CICIDS NetFlow features
- **Enrichment**: VirusTotal API for IOC correlation

## Key findings

The VQC showed competitive accuracy on small feature subsets but did not outperform gradient-boosted trees on the full feature set. Training time was significantly higher due to parameter-shift gradient estimation.

Where quantum showed promise was in capturing certain non-linear correlations in low-dimensional projections of the data — but this advantage disappeared once classical models had access to the full feature space.

## Takeaway

Quantum ML on NISQ devices is not yet practical for production IDS. But the exercise of formalizing network traffic classification as a variational optimization problem was valuable for understanding where quantum advantage might eventually appear.

*Full benchmarks, circuit diagrams, and ablation studies coming soon.*
