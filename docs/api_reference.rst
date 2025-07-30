.. _api_reference:

API Reference
=============

This section provides a high-level overview of the public C++ API of HEonGPU. The design is intended to be stable and intuitive, abstracting all underlying CUDA complexity.

.. code-block:: cpp
   :linenos:

    namespace heongpu {

        // Enum for selecting the homomorphic encryption scheme
        enum class scheme_type {
            bfv,
            ckks,
            tfhe
        };

        // Class to hold all encryption parameters
        class EncryptionParameters {
        public:
            EncryptionParameters(scheme_type scheme);
            void set_poly_modulus_degree(size_t degree);
            void set_coeff_modulus(const std::vector<Modulus>& moduli);
            //... other setters for plaintext modulus, bootstrapping mode, etc.
        };

        // The main context class for all operations
        class HEonGPUContext {
        public:
            HEonGPUContext(const EncryptionParameters& params);
            //... methods to get context data
        };

        // Generates public, secret, relinearization, and Galois keys
        class KeyGenerator {
        public:
            KeyGenerator(const HEonGPUContext& context);
            PublicKey create_public_key();
            SecretKey secret_key() const;
            RelinearizationKeys create_relin_keys();
            GaloisKeys create_galois_keys();
        };

        // Encrypts plaintexts to ciphertexts
        class Encryptor {
        public:
            Encryptor(const HEonGPUContext& context, const PublicKey& public_key);
            void encrypt(const Plaintext& ptxt, Ciphertext& ctxt);
        };

        // Decrypts ciphertexts to plaintexts
        class Decryptor {
        public:
            Decryptor(const HEonGPUContext& context, const SecretKey& secret_key);
            void decrypt(const Ciphertext& ctxt, Plaintext& ptxt);
        };

        // Performs all homomorphic evaluations on the GPU
        class Evaluator {
        public:
            Evaluator(const HEonGPUContext& context);

            // Arithmetic operations
            void add(const Ciphertext& ctxt1, const Ciphertext& ctxt2, Ciphertext& result);
            void multiply(const Ciphertext& ctxt1, const Ciphertext& ctxt2, Ciphertext& result);
            
            // Key-switching operations
            void relinearize_inplace(Ciphertext& ctxt, const RelinearizationKeys& relin_keys);
            void rotate_rows_inplace(Ciphertext& ctxt, int steps, const GaloisKeys& galois_keys);

            // CKKS-specific operations
            void rescale_to_next_inplace(Ciphertext& ctxt);

            // Bootstrapping
            void bootstrap_inplace(Ciphertext& ctxt);
        };

        //... other classes like Plaintext, Ciphertext, PublicKey, etc.
        //... and support for unsigned integer types (huint8 to huint256).
    }
