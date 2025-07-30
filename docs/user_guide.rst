.. _user_guide:

User Guide and Tutorials
========================

This section provides more detailed guidance on using the library's features, including tutorials for its supported homomorphic encryption schemes and advice on parameter selection.

Core Library Concepts
---------------------

The HEonGPU API is designed to be stable and straightforward. The main classes you will interact with are:

* ``EncryptionParameters``: Holds all parameters defining the cryptographic context.
* ``HEonGPUContext``: Manages the environment for all cryptographic operations, validating parameters and pre-computing necessary values on the GPU. Once created, the context is frozen.
* ``KeyGenerator``: Creates the public/secret key pair, as well as relinearization and Galois keys.
* ``Encryptor``, ``Decryptor``, ``Evaluator``: Perform the core cryptographic operations.
* ``ExecutionOptions``: A struct allowing the user to specify data locality (host or device) for inputs and outputs of an operation.

Tutorial: Integer Arithmetic with the BFV Scheme
------------------------------------------------

The BFV scheme is ideal for applications requiring exact computations on integers. The example in the :ref:`getting_started` guide provides a template for basic BFV operations.

Tutorial: Approximate Number Arithmetic with the CKKS Scheme
------------------------------------------------------------

The CKKS scheme is tailored for applications involving real or complex numbers, such as privacy-preserving machine learning. It performs approximate arithmetic, and managing the precision (scale) is a key consideration.

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
    //... (code to print the resulting vector)...

Parameter Selection Guide
-------------------------

Choosing the right encryption parameters is crucial for balancing security, performance, and computational depth. The library's design philosophy is for the user to explicitly configure all parameters at the ``Context`` level before any keys are generated.

* **Security Level**: The library is designed for a 128-bit security level.
* **Polynomial Modulus Degree (N)**: This parameter is the primary driver of security and performance. Larger values of N provide more security and a larger "noise budget" but at a significant performance cost. Power-of-two values for N (e.g., 4096, 8192, 16384) are recommended starting points.
* **Coefficient Modulus (q)**: This is a chain of prime numbers that determines the computational depth.
* **Plaintext Modulus (t)**: For the BFV scheme, this defines the size of the integer message space.
* **Bootstrapping Mode**: For schemes that support it (CKKS, TFHE), the specific bootstrapping mode (e.g., regular, slim, bit, or gate) must be selected during context creation.
* **Key-Switching Mode**: Settings for key-switching, such as digit decomposition, are also fixed when the context is created.

This explicit setup ensures that the user has full control and awareness of the active parameters, as the system will not silently reconfigure them.
