.. _introduction:

Introduction to HEonGPU
=======================

HEonGPU is a high-performance, open-source C++ library designed to accelerate Fully Homomorphic Encryption (FHE) operations by leveraging the immense parallel processing capabilities of modern Graphics Processing Units (GPUs) [1]_, [2]_. FHE is a transformative cryptographic technology that enables computation directly on encrypted data, offering a powerful solution for preserving data privacy in untrusted environments like cloud computing services [3]_, [4]_. However, the practical adoption of FHE has been historically hindered by its significant computational overhead, which can introduce slowdowns of up to five orders of magnitude compared to plaintext operations [4]_, [5]_. HEonGPU directly addresses this performance bottleneck, making complex, privacy-preserving applications more feasible.

The library is the work of applied cryptographer Alişah Özcan and is grounded in extensive academic research, including his master's thesis [2]_, [6]_, [7]_. Its development path shows a clear progression from an initial prototype focused on the Brakerski-Fan-Vercauteren (BFV) scheme to a more robust library that also supports the Cheon-Kim-Kim-Song (CKKS) scheme, with a clear roadmap for future expansion [1]_, [6]_. This foundation in rigorous academic research ensures that the library's performance is not merely the result of porting code to CUDA, but stems from a deep, methodical investigation into optimizing core cryptographic algorithms for GPU architectures.

Key Features
------------

HEonGPU is distinguished by a set of architectural choices and features designed for maximum performance and usability [1]_:

* **Full GPU Execution**: A core design principle of HEonGPU is that all FHE operations are executed entirely on the GPU. This architectural decision is critical for performance, as it eliminates the latency associated with frequent data transfers between the host (CPU) and the device (GPU), a common bottleneck in hybrid computational models. This makes the library ideal for applications where a complete and complex computational graph can be offloaded to the GPU.
* **Multi-Stream Architecture**: The library utilizes a multi-stream architecture, allowing for the concurrent execution of multiple CUDA kernels. This approach enhances throughput by enabling the parallel processing of independent tasks and optimizes the use of GPU memory resources, further reducing performance penalties from memory management.
* **Supported Homomorphic Encryption Schemes**: HEonGPU currently provides high-performance implementations for two of the most widely used FHE schemes:
    * **BFV Scheme**: For performing computations on encrypted integers.
    * **CKKS Scheme**: For performing approximate arithmetic on encrypted real or complex numbers, making it particularly suitable for privacy-preserving machine learning applications.
* **User-Friendly C++ Interface**: Despite the complexity of the underlying CUDA kernels, HEonGPU exposes a high-level C++ API. This interface encapsulates the GPU-specific code within easy-to-use classes, abstracting away the low-level details of GPU programming and making the library accessible to developers without specialized knowledge of CUDA.

The HEonGPU Ecosystem
---------------------

HEonGPU is a significant entry in a landscape of powerful FHE libraries that includes Microsoft SEAL, OpenFHE, HElib, and Lattigo [8]_, [9]_. It is also the cornerstone of a broader suite of GPU-accelerated cryptographic tools developed by the same author. These related projects, such as ``PIRonGPU`` (for Private Information Retrieval), ``GPU-NTT`` (a specialized Number Theoretic Transform library), ``GPU-FFT`` (for Fast Fourier Transforms), and ``RNGonGPU`` (a secure random number generator), highlight a comprehensive effort to build a high-performance, GPU-native ecosystem for privacy-enhancing technologies [2]_, [10]_. Within this ecosystem, HEonGPU serves as the central cryptographic engine.

.. [1] Özcan, A. Ş., & Savaş, E. (2024). HEonGPU: a GPU-based Fully Homomorphic Encryption Library 1.0. IACR Cryptology ePrint Archive, 2024/1543.
.. [2] Özcan, A. Ş. (2023). A GPU library for BFV homomorphic encryption scheme via three different NTT algorithms (Master's thesis, Sabanci University).
.. [3] Gentry, C. (2009). A fully homomorphic encryption scheme (Doctoral dissertation, Stanford University).
.. [4] Brakerski, Z. (2012). Fully homomorphic encryption without bootstrapping. In Proceedings of the 3rd innovations in theoretical computer science conference (pp. 309-325).
.. [5] Cheon, J. H., Kim, A., Kim, M., & Song, Y. (2017). Homomorphic encryption for arithmetic of approximate numbers. In International conference on the theory and application of cryptology and information security (pp. 409-437). Springer, Cham.
.. [6] `HEonGPU GitHub Repository <https://github.com/Alisah-Ozcan/HEonGPU>`_
.. [7] Özcan, A. Ş., Ayduman, C., Türkoğlu, E. R., & Savaş, E. (2023). Homomorphic Encryption on GPU. IEEE Access, 11, 84168-84186.
.. [8] `Microsoft SEAL <https://github.com/microsoft/SEAL>`_
.. [9] `OpenFHE <https://www.openfhe.org/>`_
.. [10] `PIRonGPU GitHub Repository <https://github.com/Alisah-Ozcan/PIRonGPU>`_
