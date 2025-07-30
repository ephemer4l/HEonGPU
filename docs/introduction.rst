.. _introduction:

Introduction to HEonGPU
=======================

HEonGPU is a high-performance, open-source C++ library designed to accelerate Fully Homomorphic Encryption (FHE) operations by leveraging the immense parallel processing capabilities of modern Graphics Processing Units (GPUs). FHE is a transformative cryptographic technology that enables computation directly on encrypted data, offering a powerful solution for preserving data privacy in untrusted environments. However, the practical adoption of FHE has been historically hindered by its significant computational overhead. HEonGPU directly addresses this performance bottleneck, making complex, privacy-preserving applications more feasible.

The library is grounded in extensive academic research, ensuring that its performance stems from a deep, methodical investigation into optimizing core cryptographic algorithms for GPU architectures.

Key Features
------------

HEonGPU is distinguished by a set of architectural choices and features designed for maximum performance and usability:

* **Full GPU Execution**: A core design principle of HEonGPU is that all FHE operations—from basic arithmetic to complex procedures like bootstrapping and key-switching—are executed entirely on the GPU. This architectural decision is critical for performance, as it eliminates the latency associated with frequent data transfers between the host (CPU) and the device (GPU).
* **Multi-Stream Architecture**: The library utilizes a multi-stream architecture, allowing for the concurrent execution of multiple CUDA kernels. This enhances throughput by enabling the parallel processing of independent tasks.
* **Broad Scheme Support**: HEonGPU provides high-performance implementations for several widely used FHE schemes:
    * **BFV**: For exact computations on encrypted integers.
    * **CKKS**: For approximate arithmetic on encrypted real or complex numbers.
    * **TFHE**: For boolean circuit evaluation and programmable bootstrapping.
* **Advanced Bootstrapping**: Full bootstrapping support is a cornerstone feature, enabling computations of arbitrary depth. Implementations include:
    * **CKKS**: Regular, slim, bit, and gate bootstrapping.
    * **TFHE**: Gate bootstrapping.
* **User-Friendly C++ Interface**: Despite the complexity of the underlying CUDA kernels, HEonGPU exposes a high-level C++ API that abstracts away low-level details, making the library accessible to developers without specialized knowledge of CUDA.

The HEonGPU Ecosystem
---------------------

HEonGPU is a significant entry in a landscape of powerful FHE libraries and is the cornerstone of a broader suite of GPU-accelerated cryptographic tools developed by the same author. These related projects, such as ``PIRonGPU``, ``GPU-NTT``, ``GPU-FFT``, and ``RNGonGPU``, highlight a comprehensive effort to build a high-performance, GPU-native ecosystem for privacy-enhancing technologies.
