Installation
============

You can install PyOMP from PyPI using `pip`:

.. code-block:: console

   $ pip install pyomp

It is also possible to install PyOMP through `conda`:

.. code-block:: console

   $ conda install -c python-for-hpc -c conda-forge pyomp

Compatibility
-------------

PyOMP releases are compatible with a specific range of Numba versions. The table
below summarizes the supported Numba versions for each PyOMP release series.
The `x` in versions indicates that all patch levels within that version are
supported.

+--------+---------------------+
| PyOMP  | Numba               |
+========+=====================+
| 0.5.x  | 0.62.x - 0.63.x     |
+--------+---------------------+
| 0.4.x  | 0.61.x              |
+--------+---------------------+
| 0.3.x  | 0.57.x - 0.60.x     |
+--------+---------------------+

LLVM 22 source build
--------------------

The LLVM 22 development branch requires Numba 0.66.x (llvmlite 0.48.x).
Build with Clang and LLVM development libraries version 22.1.8, including
LLVM's NVPTX and AMDGPU targets and the LLVM linker tools. On Linux, install
the libffi and libelf development headers and the ``patch`` utility as well.
Use an isolated environment:

.. code-block:: console

   $ python3 -m venv .venv22
   $ . .venv22/bin/activate
   $ pip install setuptools setuptools-scm wheel cmake ninja 'numba==0.66.0' lark cffi
   $ export LLVM_VERSION=22.1.8 LLVM_DIR=$(llvm-config --cmakedir)
   $ export CC=clang CXX=clang++ CMAKE_BUILD_PARALLEL_LEVEL=4
   $ export ENABLE_BUNDLED_LIBOMP=1 ENABLE_BUNDLED_LIBOMPTARGET=1
   $ pip wheel --no-build-isolation --no-deps . -w dist
   $ pip install --no-deps dist/pyomp-*.whl
   $ RUN_TARGET=0 python -m numba.runtests -v -- numba.openmp.tests.test_openmp
   $ OMP_TARGET_OFFLOAD=mandatory TEST_DEVICE=host RUN_TARGET=1 python -m numba.runtests -v -- numba.openmp.tests.test_openmp.TestOpenmpTarget

The build downloads LLVM sources and applies the versioned runtime patches.
LLVM 22 builds GPU device bitcode separately from libomptarget; both NVPTX
and AMDGPU bitcode are included in the Linux wheel. Set
``ENABLE_BUNDLED_LIBOMP=0`` when using an existing LLVM OpenMP runtime, as in
the Conda recipe. ``CMAKE_BUILD_PARALLEL_LEVEL`` controls build concurrency.

Wheel and Conda CPU tests pass on Linux x86_64, Linux ARM64, and macOS ARM64 with Python
3.10–3.14 and Numba 0.66.0. On Linux, the host suite runs 235 tests with
120 passing and 115 existing skips, including disabled target tests and
unsupported clauses. The separate mandatory host-device run passes 68 tests
with 4 existing skips. On macOS, the host suite passes 119 tests with 116 skips;
all 72 target tests are skipped because offloading is unsupported there.
Local validation also passes with Python 3.12, llvmlite 0.48.0 (LLVM 22.1.0),
and Clang/runtime 22.1.8.
Four basic GPU tests also pass on an NVIDIA RTX 3080 (sm_86), driver 595.84,
using CUDA 12.8.93 compiler tools: teams/distribute/parallel-for, tofrom
mapping, explicit updates, and a parallel reduction. These use mandatory
offload and select the NVIDIA device; CPU fallback is not counted as success.
The full GPU suite, other GPUs (including Blackwell), CUDA 13 compiler tools,
and other operating systems and architectures remain unverified.
This update does not add target ``nowait``/``depend`` support.

To repeat the basic NVIDIA checks with an installed wheel, expose the NVIDIA
driver to the process and provide CUDA compiler tools. The isolated test
used NVIDIA's pip package with the following paths:

.. code-block:: console

   $ pip install nvidia-cuda-nvcc-cu12==12.8.93
   $ export CUDA_HOME="$VIRTUAL_ENV/lib/python3.12/site-packages/nvidia/cuda_nvcc"
   $ export PATH="$CUDA_HOME/bin:$PATH"
   $ export LD_LIBRARY_PATH="$CUDA_HOME/nvvm/lib64${LD_LIBRARY_PATH:+:$LD_LIBRARY_PATH}"
   $ export OMP_TARGET_OFFLOAD=mandatory TEST_DEVICE=gpu RUN_TARGET=1
   $ python -m numba.runtests -v -- \
       numba.openmp.tests.test_openmp.TestOpenmpTarget.test_target_teams_distribute_parallel_for \
       numba.openmp.tests.test_openmp.TestOpenmpTarget.test_target_data_tofrom \
       numba.openmp.tests.test_openmp.TestOpenmpTarget.test_target_update_to_from \
       numba.openmp.tests.test_openmp.TestOpenmpTarget.test_target_teams_distribute_parallel_for_reduction

Additional options
------------------

Binder and Docker images are provided to try PyOMP without installing locally.

Binder (free hosted JupyterLab):
`Binder <https://mybinder.org/v2/gh/Python-for-HPC/binder/HEAD>`_

Docker (pre-built images):

.. code-block:: console

   $ docker pull ghcr.io/python-for-hpc/pyomp:latest
