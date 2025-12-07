# XC GPU on AWS ⚡

Dedicated resources for running **NVIDIA GPU** workloads on AWS Accelerated Computing instances (P5, P4d, G5, etc.).

## 🎯 Objectives
* **Environment Setup:** Configuring NVIDIA drivers, CUDA, and EFA on AWS.
* **Distributed Training:** Multi-node training using NCCL and EFA.
* **Frameworks:** Best practices for PyTorch, NeMo, and Megatron-LM on AWS.
* **Inference:** High-performance serving with TensorRT-LLM and Triton.

## 📂 Contents
* `/efa-setup`: Elastic Fabric Adapter (EFA) configuration and verification (NCCL tests).
* `/multi-node-training`: Slurm scripts and guides for P-series instances.
* `/inference-optimization`: Optimizing throughput and latency on GPU instances.

## 🔗 Related Repositories
* [XC Common Infra](https://github.com/your-username/xc-common-infra-aws)
* [ParallelCluster on AWS](https://github.com/your-username/parallel-cluster-on-aws)
