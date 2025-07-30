.. _advanced_topics:

Advanced Topics
===============

This section covers advanced usage patterns and integrations with other libraries.

Interoperability with Microsoft SEAL
------------------------------------

A key strategic aspect of HEonGPU is its designed interoperability with Microsoft SEAL. This positions HEonGPU not just as a standalone solution, but as a potential GPU acceleration backend for the broader FHE ecosystem.

Advanced users can leverage this by using SEAL for its robust API for parameter selection, encoding, and key management on the CPU, while replacing the performance-critical calls to SEAL's ``Evaluator`` with calls to HEonGPU's highly optimized GPU-based ``Evaluator``. This allows developers to gain massive performance improvements without needing to rewrite their entire application logic.

Building Dependent Projects
---------------------------

The library is designed for clean integration into larger projects using standard CMake mechanisms. Projects like ``PIRonGPU`` demonstrate this maturity. A typical ``CMakeLists.txt`` for a dependent project would include:

.. code-block:: cmake

    # Find the HEonGPU package installed on the system
    find_package(HEonGPU 1.1.0 REQUIRED)

    # Add the executable for your application
    add_executable(my_app main.cpp)

    # Link your application against the HEonGPU library
    # This automatically handles include directories and library paths.
    target_link_libraries(my_app PRIVATE HEonGPU::heongpu)
