.. _examples-tools-cmake-toolchain-intel-oneapi:

CMakeToolchain: Using Intel OneAPI compiler
===========================================

Conan has integrations and can work with Intel OneAPI compiler

Let's start from a simple ``cmake_lib`` template:

.. code-block:: bash

    $ conan new cmake_lib

This creates a simple CMake based project and Conan package recipe that uses ``CMakeToolchain``.


Intel OneAPI in Windows (MSVC backend)
--------------------------------------

To build this configuration we will use the following profile:

.. code-block:: ini
    :caption: intel_win

    [settings]
    os=Windows
    compiler=intel-cc
    compiler.version=2025.1
    compiler.mode=dpcpp
    compiler.runtime=dynamic
    compiler.cppstd=14
    arch=x86_64
    build_type=Release

    [conf]
    tools.intel:installation_path=C:\Program Files (x86)\Intel\oneAPI
    tools.cmake.cmaketoolchain:generator=Ninja

    [tool_requires]
    ninja/[*]

Quick explanation of the profile:

- The ``compiler.runtime`` definition is the important differentiator to distinguish between the Linux gcc-backend and
  the Windows backend with the MSVC runtime. When using ``compiler.runtime`` it refers to the MSVC runtime (static or dynamic).
- The ``intel-cc`` compiler has different modes like ``"icx", "classic", "dpcpp"``, one of them must be selected.
- It is necessary to specify ``tools.intel:installation_path``, so the installation can be found.
- We are using the ``Ninja`` CMake generator, and installing it from a ``[tool_requires]``, but this might not be necessary if Ninja
  is installed in your system.


Let's build it:

.. code-block:: bash

    $ conan build . -pr=intel_win

    ======== Input profiles ========
    Profile host:
    [settings]
    arch=x86_64
    build_type=Release
    compiler=intel-cc
    compiler.cppstd=14
    compiler.mode=dpcpp
    compiler.runtime=dynamic
    compiler.version=2025.1
    os=Windows

    [tool_requires]
    *: ninja/[*]

    [conf]
    tools.cmake.cmaketoolchain:generator=Ninja
    tools.intel:installation_path=C:\Program Files (x86)\Intel\oneAPI

    ...

    ======== Calling build() ========
    conanfile.py (mypkg/0.1): Calling build()
    conanfile.py (mypkg/0.1): Running CMake.configure()
    conanfile.py (mypkg/0.1): RUN: cmake -G "Ninja" -DCMAKE_TOOLCHAIN_FILE="generators/conan_toolchain.cmake" -DCMAKE_BUILD_TYPE="Release" 
    :: initializing oneAPI environment...
    Initializing Visual Studio command-line environment...
    Visual Studio version 18.1.1 environment configured.
    "C:\Program Files\Microsoft Visual Studio\18\Community\"
    Visual Studio command-line environment initialized for: 'x64'
    :: oneAPI environment initialized ::
    -- Using Conan toolchain: ... conan_toolchain.cmake
    -- Conan toolchain: Defining architecture flag: /Qm64
    -- The CXX compiler identification is IntelLLVM 2026.0.0 with MSVC-like command-line
    -- Check for working CXX compiler: C:/Program Files (x86)/Intel/oneAPI/compiler/latest/bin/icx.exe - skipped

    conanfile.py (mypkg/0.1): Running CMake.build()
    conanfile.py (mypkg/0.1): RUN: cmake --build "C:\Users\Diego\conanws\kk\build\Release" -- -j8
    :: initializing oneAPI environment...
    Initializing Visual Studio command-line environment...
    Visual Studio version 18.1.1 environment configured.
    "C:\Program Files\Microsoft Visual Studio\18\Community\"
    Visual Studio command-line environment initialized for: 'x64'
    :: oneAPI environment initialized ::
    [3/3] Linking CXX executable mypkg.exe


We can see that the Intel ``setvars.bat`` environment is activated (this configures the compiler environment in the same way as ``vcvars.bat`` does
for the ``msvc`` compiler). This is implemented by the :ref:`IntelCC generator<reference_tools_intel>`

We can run the created executable and see both some traces of ``MSVC`` compiler, like the MSVC dynamic runtime, but also some ``clang`` related flags.

.. code-block:: bash

    $ mypkg/0.1: Hello World Release!
        mypkg/0.1: _M_X64 defined
        mypkg/0.1: __x86_64__ defined
        mypkg/0.1: MSVC runtime: MultiThreadedDLL
        mypkg/0.1: _MSC_VER1950
        mypkg/0.1: _MSVC_LANG201402
        mypkg/0.1: __cplusplus201402
        mypkg/0.1: __clang_major__22
        mypkg/0.1: __clang_minor__1
    mypkg/0.1 test_package


.. seealso::

    - :ref:`IntelCC generator<reference_tools_intel>`