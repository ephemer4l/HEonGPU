.. _user_guide:

User Guide and Tutorials
========================

This section provides more detailed guidance on using the library's features, including tutorials for its supported homomorphic encryption schemes and advice on parameter selection.

Core Library Concepts
---------------------

The HEonGPU API is designed to be intuitive, especially for users familiar with other FHE libraries like Microsoft SEAL. The main classes you will interact with are:

* ``EncryptionParameters``: Holds all the parameters defining the cryptographic context, including the scheme type (BFV or CKKS), polynomial modulus degree, and coefficient moduli.
* ``HEonGPUContext``: Manages the environment for all cryptographic operations, validating parameters and pre-computing necessary values on the GPU.
* ``KeyGenerator``: Creates the public and secret key pair, as well as relinearization and Galois keys needed for specific homomorphic operations.
* ``Encryptor``: Encrypts plaintext data into ciphertexts using the public key.
* ``Decryptor``: Decrypts ciphertexts back into plaintexts using the secret key.
* ``Evaluator``: The workhorse of the library, performing all homomorphic operations (e.g., addition, multiplication, relinearization, rotation) on ciphertexts entirely on the GPU.

Tutorial: Integer Arithmetic with the BFV Scheme
------------------------------------------------

The BFV scheme is ideal for applications requiring exact computations on integers, such as secure database queries or statistical analysis.

.. code-block:: cpp
   :linenos:

    //... (setup and key generation as in the previous example)...

    // Encrypt two integers
    int64_t a = 12;
    int64_t b = -5;
    heongpu::Plaintext ptxt_a(std::to_string(a));
    heongpu::Plaintext ptxt_b(std::to_string(b));
    heongpu::Ciphertext ctxt_a, ctxt_b;
    encryptor.encrypt(ptxt_a, ctxt_a);
    encryptor.encrypt(ptxt_b, ctxt_b);

    // Perform homomorphic multiplication
    heongpu::Ciphertext ctxt_mul_result;
    evaluator.multiply(ctxt_a, ctxt_b, ctxt_mul_result);

    // Multiplication increases the size of the ciphertext.
    // Relinearization is needed to reduce it back to the original size.
    heongpu::RelinearizationKeys relin_keys = keygen.create_relin_keys();
    evaluator.relinearize_inplace(ctxt_mul_result, relin_keys);

    // Decrypt and verify
    heongpu::Plaintext ptxt_mul_result;
    decryptor.decrypt(ctxt_mul_result, ptxt_mul_result);
    //... (code to print and verify result, which should be -60)...

Tutorial: Approximate Number Arithmetic with the CKKS Scheme
------------------------------------------------------------

The CKKS scheme is tailored for applications involving real or complex numbers, such as privacy-preserving machine learning inference. It performs approximate arithmetic, and managing the precision (scale) of the encrypted values is a key consideration.

.. code-block:: cpp
   :linenos:

    // 1. Set up parameters for the CKKS scheme
    heongpu::EncryptionParameters params(heongpu::scheme_type::ckks);
    size_t poly_modulus_degree = 8192;
    params.set_poly_modulus_degree(poly_modulus_degree);
    params.set_coeff_modulus(heongpu::CoeffModulus::Create(poly_modulus_degree, {60, 40, 40, 60}));

    //... (context and key generation similar to BFV)...

    // 2. Create an encoder for CKKS
    double initial_scale = pow(2.0, 40);
    heongpu::CKKSEncoder encoder(context);

    // 3. Encode and encrypt a vector of doubles
    std::vector<double> input_vec = {3.1415, 2.7182, 1.6180};
    heongpu::Plaintext ptxt_vec;
    encoder.encode(input_vec, initial_scale, ptxt_vec);
    heongpu::Ciphertext ctxt_vec;
    encryptor.encrypt(ptxt_vec, ctxt_vec);

    // 4. Perform a homomorphic operation (e.g., square the vector)
    heongpu::Ciphertext ctxt_squared;
    evaluator.square(ctxt_vec, ctxt_squared);
    evaluator.relinearize_inplace(ctxt_squared, relin_keys);

    // After multiplication, the scale of the ciphertext also squares.
    // We need to rescale it back down to maintain precision.
    evaluator.rescale_to_next_inplace(ctxt_squared);

    // 5. Decrypt and decode the result
    heongpu::Plaintext ptxt_squared;
    decryptor.decrypt(ctxt_squared, ptxt_squared);
    std::vector<double> output_vec;
    encoder.decode(ptxt_squared, output_vec);
    //... (code to print the resulting vector, which will be approximately {9.869, 7.388, 2.618})...

Parameter Selection Guide
-------------------------

Choosing the right encryption parameters is crucial for balancing security, performance, and computational depth.

* **Security Level**: The library was developed and tested for a 128-bit security level, which is the standard for most modern applications.
* **Polynomial Modulus Degree (N)**: This parameter is the primary driver of security and performance. Larger values of N provide more security and a larger "noise budget" (allowing for more computations), but at a significant performance cost. The prototype versions of the library focused on power-of-two values for N such as 4096, 8192, 16384, and 32768. These are recommended starting points.
* **Coefficient Modulus (q)**: This is a chain of prime numbers. The size and number of these primes determine the computational depth of the circuit you can evaluate. A larger total bit-size for q allows for more sequential multiplications before the noise overwhelms the signal. The library's use of the Residue Number System (RNS) makes operations with multiple coefficient moduli highly suitable for parallelization on the GPU.

For newcomers, it is highly recommended to start with the pre-defined parameter sets available in the library or to use the parameters from the tutorial examples.
