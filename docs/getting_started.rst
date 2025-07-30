.. _getting_started:

Getting Started
===============

This section provides a practical guide to installing HEonGPU and running a basic encrypted computation. The process is designed to be straightforward for developers familiar with C++ and CMake.

Prerequisites
-------------

Before building the library, ensure your development environment meets the following requirements:

* **CMake**: Version 3.26.4 or higher.
* **C++ Compiler**: A modern C++ compiler with C++17 support (e.g., GCC).
* **NVIDIA CUDA Toolkit**: Version 11.4 or higher.

Building the Library
--------------------

The library uses CMake for a cross-platform build process. The most critical configuration step is specifying the correct compute capability for your target GPU.

1.  **Clone the Repository**:
    First, obtain the source code from the official GitHub repository.

    .. code-block:: bash

        git clone https://github.com/Alisah-Ozcan/HEonGPU.git
        cd HEonGPU

2.  **Configure with CMake**:
    Run CMake from the root of the repository to generate the build files. You must set the ``CMAKE_CUDA_ARCHITECTURES`` variable to match your GPU's architecture. This flag instructs the CUDA compiler (``nvcc``) to generate code specifically optimized for your hardware, which is essential for both correctness and performance.

    .. code-block:: bash

        cmake -S. -D CMAKE_CUDA_ARCHITECTURES=XX -B build

    Replace ``XX`` with the appropriate value from the table below. For example, for an NVIDIA A100 GPU, you would use ``80``.

    .. list-table:: GPU Architecture to CMAKE_CUDA_ARCHITECTURES Mapping
       :widths: 25 25
       :header-rows: 1

       * - GPU Architecture
         - ``CMAKE_CUDA_ARCHITECTURES`` Value
       * - Volta
         - 70, 72
       * - Turing
         - 75
       * - Ampere
         - 80, 86
       * - Ada Lovelace
         - 89, 90

3.  **Compile the Library**:
    Once configuration is complete, build the library using the following command:

    .. code-block:: bash

        cmake --build ./build/ --config Release -j

    This will compile the HEonGPU static or shared library, which will be located in the ``build/lib`` directory.

Your First Encrypted Computation
---------------------------------

After successfully building the library, you can write a simple program to verify its functionality. The following conceptual example demonstrates a typical FHE workflow: setup, key generation, encryption, computation, and decryption.

.. code-block:: cpp
   :linenos:

    // Note: This is a conceptual example. Actual class and method names may vary.
    #include <iostream>
    #include "heongpu/heongpu.h" // Assumed header

    int main() {
        // 1. Set up encryption parameters for the BFV scheme
        heongpu::EncryptionParameters params(heongpu::scheme_type::bfv);
        size_t poly_modulus_degree = 8192;
        params.set_poly_modulus_degree(poly_modulus_degree);
        params.set_coeff_modulus(heongpu::CoeffModulus::BFVDefault(poly_modulus_degree));
        params.set_plain_modulus(1024);

        // 2. Create the HEonGPU context
        auto context = heongpu::HEonGPUContext(params);

        // 3. Generate keys
        heongpu::KeyGenerator keygen(context);
        heongpu::PublicKey public_key = keygen.create_public_key();
        heongpu::SecretKey secret_key = keygen.secret_key();

        // 4. Create Encryptor and Evaluator objects
        heongpu::Encryptor encryptor(context, public_key);
        heongpu::Evaluator evaluator(context);
        heongpu::Decryptor decryptor(context, secret_key);

        // 5. Encrypt data
        int64_t x = 6;
        int64_t y = 7;
        heongpu::Plaintext ptxt_x(std::to_string(x));
        heongpu::Plaintext ptxt_y(std::to_string(y));

        heongpu::Ciphertext ctxt_x, ctxt_y;
        encryptor.encrypt(ptxt_x, ctxt_x);
        encryptor.encrypt(ptxt_y, ctxt_y);

        // 6. Perform homomorphic addition
        heongpu::Ciphertext ctxt_result;
        evaluator.add(ctxt_x, ctxt_y, ctxt_result);

        // 7. Decrypt the result
        heongpu::Plaintext ptxt_result;
        decryptor.decrypt(ctxt_result, ptxt_result);

        // 8. Verify the result
        int64_t result;
        std::from_chars(ptxt_result.to_string().data(), ptxt_result.to_string().data() + ptxt_result.to_string().size(), result);
        std::cout << "Plaintext values: x = " << x << ", y = " << y << std::endl;
        std::cout << "Homomorphic addition result: " << result << std::endl;

        return 0;
    }
