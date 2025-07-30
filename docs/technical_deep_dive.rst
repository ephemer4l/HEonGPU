.. _technical_deep_dive:

Technical Deep Dive
===================

This section explores the advanced technical details of HEonGPU, focusing on the architectural decisions and optimization strategies that enable its high performance.

Library Architecture
--------------------

The performance of HEonGPU is rooted in its "all-in-on-the-GPU" design philosophy. Unlike some libraries that may offload only the most intensive kernels, HEonGPU is designed to keep the entire computational workflow on the GPU. This strategy is a direct response to a critical observation: for many FHE workloads, the system is not purely compute-bound but becomes memory-bound due to the large size of ciphertexts. The latency of transferring these large data structures between CPU and GPU memory can negate the speedup gained from GPU computation. By keeping data resident on the GPU, HEonGPU minimizes this costly overhead.

This architecture is complemented by the use of multiple CUDA streams, which allows the GPU's scheduler to overlap memory transfers with computation and execute independent kernels concurrently, maximizing hardware utilization.

Accelerating Polynomial Arithmetic: The Number Theoretic Transform (NTT)
------------------------------------------------------------------------

The most computationally intensive operation in lattice-based FHE schemes like BFV and CKKS is the multiplication of large-degree polynomials. HEonGPU accelerates this using the Number Theoretic Transform (NTT), which allows two polynomials to be multiplied in quasi-linear time, :math:`O(N \log N)`.

The research behind HEonGPU involved a systematic investigation into the most efficient GPU implementations of the NTT, comparing several algorithms:

* **MERGE-NTT**: Based on the recursive Cooley-Tukey FFT algorithm, an improved version, ``MERGE-2``, overcomes register memory limitations by intelligently partitioning the algorithm's outer loops into multiple CUDA kernels.
* **4STEP-NTT**: This algorithm reframes the 1D NTT as a series of smaller 2D NTTs. While this can be efficient, it introduces costly matrix transpose operations, which were carefully optimized in the HEonGPU implementation.

The research concluded that a carefully tuned ``MERGE-2`` implementation generally provided the best performance.

Advanced GPU Optimization Strategies
------------------------------------

The speed of HEonGPU is due to a meticulous application of GPU-specific optimization techniques:

* **Minimizing Global Memory Access**: Implementations are designed to reduce kernel calls and limit reads/writes to the GPU's slower global memory.
* **Coalesced Memory Access**: Kernels are structured so that threads within a warp access consecutive memory locations, maximizing effective bandwidth.
* **Efficient Shared Memory Use**: Fast, on-chip shared memory is used as a programmable cache to stage data for computations.
* **Tuning CUDA Launch Parameters**: The library's development involved creating a "recipe" for selecting the optimal CUDA launch parameters (kernel count, block size, grid shape) for a given polynomial degree and GPU architecture.
* **Optimized Modular Reduction**: The library uses highly optimized modular reduction algorithms, including Barrett, Plantard, and a particularly efficient Goldilocks reduction for 64-bit primes, implemented directly in assembly for maximum speed.

Implementing Homomorphic Operations as CUDA Kernels
--------------------------------------------------

A complex homomorphic operation, such as a full BFV ciphertext multiplication, is decomposed into a sequence of distinct CUDA kernels. This modular approach allows each step to be individually optimized. For example, a multiplication involves:

1.  Kernels for base conversion (part of the RNS machinery).
2.  An NTT kernel to transform ciphertexts into the evaluation domain.
3.  A simple kernel for fast element-wise multiplication of the transformed polynomials.
4.  An inverse NTT (INTT) kernel to transform the result back to the coefficient domain.
5.  Additional kernels for relinearization, which itself involves more NTTs and base conversions.
