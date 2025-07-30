.. _technical_deep_dive:

Technical Deep Dive
===================

This section explores the advanced technical details of HEonGPU, focusing on the architectural decisions and optimization strategies that enable its high performance. The general architecture is described in detail in [ePrint 2024/1543](https://eprint.iacr.org/2024/1543).

Library Architecture
--------------------

The performance of HEonGPU is rooted in its "all-in-on-the-GPU" design philosophy. Unlike libraries that may offload only the most intensive kernels, HEonGPU is designed to keep the entire computational workflow on the GPU. This includes not only basic arithmetic but also complex procedures like **bootstrapping** and **key-switching**. This strategy is a direct response to a critical observation: for many FHE workloads, the system is not purely compute-bound but becomes memory-bound due to the large size of ciphertexts. The latency of transferring these large data structures between CPU and GPU memory can negate the speedup gained from GPU computation.

This architecture is complemented by the use of multiple CUDA streams, which allows the GPU's scheduler to overlap memory transfers with computation and execute independent kernels concurrently, maximizing hardware utilization.

Accelerating Polynomial Arithmetic: The Number Theoretic Transform (NTT)
------------------------------------------------------------------------

The most computationally intensive operation in lattice-based FHE is the multiplication of large-degree polynomials. HEonGPU accelerates this using the Number Theoretic Transform (NTT), which allows two polynomials to be multiplied in quasi-linear time, :math:`O(N \log N)`. The research behind HEonGPU involved a systematic investigation into the most efficient GPU implementations of the NTT, comparing several algorithms and concluding that a carefully tuned ``MERGE-2`` implementation generally provided the best performance.

Bootstrapping on the GPU
------------------------

Bootstrapping is a critical FHE technique that "refreshes" a ciphertext, resetting its noise level to allow for continued computation. This enables circuits of arbitrary depth. In HEonGPU, bootstrapping is implemented entirely on the GPU and is highly optimized.

* **CKKS Bootstrapping**: The library supports multiple variants of CKKS bootstrapping:
    * **Regular**: A full-precision bootstrapping operation.
    * **Slim**: A faster variant with slightly different properties.
    * **Bit and Gate**: Extremely fast, specialized variants for boolean logic, with amortized execution times in the microseconds per slot.
* **TFHE Bootstrapping**: Gate bootstrapping is supported for the TFHE scheme, enabling the evaluation of boolean circuits.

A paper detailing the specifics of the CKKS bootstrapping implementation is currently in preparation.

Advanced GPU Optimization Strategies
------------------------------------

The speed of HEonGPU is due to a meticulous application of GPU-specific optimization techniques:

* **Minimizing Global Memory Access**: Implementations are designed to reduce kernel calls and limit reads/writes to the GPU's slower global memory.
* **Coalesced Memory Access**: Kernels are structured so that threads within a warp access consecutive memory locations, maximizing effective bandwidth.
* **Efficient Shared Memory Use**: Fast, on-chip shared memory is used as a programmable cache to stage data for computations.
* **Optimized Modular Reduction**: The library uses highly optimized modular reduction algorithms, including Barrett, Plantard, and Goldilocks reduction, implemented directly in assembly for maximum speed.
* **Optimized Key-Switching**: Key-switching is another core primitive that has been heavily optimized. The techniques are detailed in [ePrint 2025/124](https://eprint.iacr.org/2025/124).
