.. _appendix:

Appendix
========

Performance Benchmarks
----------------------

The performance of HEonGPU has been extensively benchmarked, demonstrating significant speedups over CPU-based libraries. Performance is highly dependent on the chosen parameters and hardware. All benchmarks below are from the project's `README <https://github.com/Alisah-Ozcan/HEonGPU#readme>`_.

.. list-table:: CKKS Bootstrapping Performance (N = 2^16, on NVIDIA RTX 4090)
   :widths: 50 50
   :header-rows: 1

   * - Bootstrapping Variant
     - Execution Time
   * - Regular
     - < 170 ms
   * - Bit / Gate (amortized)
     - ~2-3 µs per slot

The TFHE implementation for unsigned integer arithmetic has also been shown to outperform other known CUDA-based implementations. For detailed timings on other operations like addition, multiplication, and relinearization, please refer to the publications.

List of Publications
--------------------

The development of HEonGPU is backed by rigorous academic research.

1.  Özcan, A. Ş., & Savaş, E. (2024). **HEonGPU: a GPU-based Fully Homomorphic Encryption Library 1.0**. *IACR Cryptology ePrint Archive, 2024/1543*.
2.  Özcan, A. Ş., & Savaş, E. (2025). **GPU Implementations of Three Different Key-Switching Methods for Homomorphic Encryption Schemes**. *IACR Cryptology ePrint Archive, 2025/124*.
3.  Özcan, A. Ş., Ayduman, C., Türkoğlu, E. R., & Savaş, E. (2023). **Homomorphic Encryption on GPU**. *IEEE Access, 11*, 84168-84186.
4.  Özcan, A. Ş. (2023). **A GPU library for BFV homomorphic encryption scheme via three different NTT algorithms** (Master's thesis, Sabanci University).

