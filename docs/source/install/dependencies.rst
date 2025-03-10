.. _install-dependencies:

Dependencies
============

ImpactX depends on the following popular third party software.
Please see installation instructions below.

- a mature `C++17 <https://en.wikipedia.org/wiki/C%2B%2B17>`__ compiler, e.g., GCC 8.4+, Clang 7, NVCC 11.0, MSVC 19.15 or newer
- `CMake 3.20.0+ <https://cmake.org>`__
- `Git 2.18+ <https://git-scm.com>`__
- `AMReX <https://amrex-codes.github.io>`__: we automatically download and compile a copy
- `ABLASTR/WarpX <https://github.com/ECP-WarpX/warpx>`__: we automatically download and compile a copy

Optional dependencies include:

- `MPI 3.0+ <https://www.mpi-forum.org/docs/>`__: for multi-node and/or multi-GPU execution
- for on-node accelerated compute *one of either*:

  - `OpenMP 3.1+ <https://www.openmp.org>`__: for threaded CPU execution or
  - `CUDA Toolkit 11.0+ (11.3+ recommended) <https://developer.nvidia.com/cuda-downloads>`__: for Nvidia GPU support (see `matching host-compilers <https://gist.github.com/ax3l/9489132>`_) or
  - `ROCm 5.2+ (5.5+ recommended) <https://gpuopen.com/learn/amd-lab-notes/amd-lab-notes-rocm-installation-readme/>`__: for AMD GPU support
- `FFTW3 <http://www.fftw.org>`__: for algorithms such as IGF space charge solver or CSR when running on CPU or with SYCL

  - also needs the ``pkg-config`` tool on Unix
- `openPMD-api 0.15.2+ <https://github.com/openPMD/openPMD-api>`__: we automatically download and compile a copy of openPMD-api for openPMD I/O support

  - see `optional I/O backends <https://github.com/openPMD/openPMD-api#dependencies>`__
- `CCache <https://ccache.dev>`__: to speed up rebuilds (For CUDA support, needs version 3.7.9+ and 4.2+ is recommended)
- `Ninja <https://ninja-build.org>`__: for faster parallel compiles
- `Python 3.8+ <https://www.python.org>`__

  - `mpi4py <https://mpi4py.readthedocs.io>`__
  - `numpy <https://numpy.org>`__
  - `quantiphy <https://quantiphy.readthedocs.io/>`__
  - `openPMD-api <https://github.com/openPMD/openPMD-api>`__
  - see our ``requirements.txt`` file for compatible versions

If you are on a high-performance computing (HPC) system, then :ref:`please see our separate HPC documentation <install-hpc>`.

For all other systems, we recommend to use a **package dependency manager**:
Pick *one* of the installation methods below to install all dependencies for ImpactX development in a consistent manner.


Conda (Linux/macOS/Windows)
---------------------------

`Conda <https://conda.io>`__/`Mamba <https://mamba.readthedocs.io>`__ are cross-compatible, user-level package managers.

.. tip::

   We recommend to configure your conda to use the faster ``libmamba`` `dependency solver <https://www.anaconda.com/blog/a-faster-conda-for-a-growing-community>`__.

   .. code-block:: bash

      conda update -y -n base conda
      conda install -y -n base conda-libmamba-solver
      conda config --set solver libmamba

   We recommend to deactivate that conda self-activates its ``base`` environment.
   This `avoids interference with the system and other package managers <https://collegeville.github.io/CW20/WorkshopResources/WhitePapers/huebl-working-with-multiple-pkg-mgrs.pdf>`__.

   .. code-block:: bash

      conda config --set auto_activate_base false

.. tab-set::

   .. tab-item:: With MPI (only Linux/macOS)

      .. code-block:: bash

         conda create -n impactx-cpu-mpich-dev -c conda-forge blaspp boost ccache cmake compilers git lapackpp "openpmd-api=*=mpi_mpich*" python numpy pandas quantiphy scipy yt "fftw=*=mpi_mpich*" pkg-config matplotlib mamba ninja mpich pip virtualenv
         conda activate impactx-cpu-mpich-dev

         # compile ImpactX with -DImpactX_MPI=ON
         # for pip, use: export IMPACTX_MPI=ON

   .. tab-item:: Without MPI

      .. code-block:: bash

         conda create -n impactx-cpu-dev -c conda-forge blaspp boost ccache cmake compilers git lapackpp openpmd-api python numpy pandas quantiphy scipy yt fftw pkg-config matplotlib mamba ninja pip virtualenv
         conda activate impactx-cpu-dev

         # compile ImpactX with -DImpactX_MPI=OFF
         # for pip, use: export IMPACTX_MPI=OFF

For OpenMP support, you will further need:

.. tab-set::

   .. tab-item:: Linux

      .. code-block:: bash

         conda install -c conda-forge libgomp

   .. tab-item:: macOS or Windows

      .. code-block:: bash

         conda install -c conda-forge llvm-openmp


Spack (Linux/macOS)
-------------------

`Spack <https://spack.readthedocs.io>`__ is a user-level package manager.
It is primarily written for Linux, with slightly less support for macOS, and future support for Windows.

First, download a `WarpX Spack desktop development environment <https://github.com/ECP-WarpX/WarpX/blob/development/Tools/machines/desktop>`__ of your choice (which will also work for ImpactX).
For most desktop developments, pick the OpenMP environment for CPUs unless you have a supported GPU.

* **Debian/Ubuntu** Linux:

  * OpenMP: ``system=ubuntu; compute=openmp`` (CPUs)
  * CUDA: ``system=ubuntu; compute=cuda`` (Nvidia GPUs)
  * ROCm: ``system=ubuntu; compute=rocm`` (AMD GPUs)
  * SYCL: *todo* (Intel GPUs)
* **macOS**: first, prepare with ``brew install gpg2; brew install gcc``

  * OpenMP: ``system=macos; compute=openmp``

If you already `installed Spack <https://spack.io>`__, we recommend to activate its `binary caches <https://spack.io/spack-binary-packages/>`__ for faster builds:

.. code-block:: bash

   spack mirror add rolling https://binaries.spack.io/develop
   spack buildcache keys --install --trust

Now install the WarpX/ImpactX dependencies in a new development environment:

.. code-block:: bash

   # download environment file
   curl -sLO https://raw.githubusercontent.com/ECP-WarpX/WarpX/development/Tools/machines/desktop/spack-${system}-${compute}.yaml

   # create new development environment
   spack env create impactx-${compute}-dev spack-${system}-${compute}.yaml
   spack env activate impactx-${compute}-dev

   # installation
   spack install
   python3 -m pip install jupyter matplotlib numpy openpmd-api openpmd-viewer pandas quantiphy scipy virtualenv yt

In new terminal sessions, re-activate the environment with

.. code-block:: bash

   spack env activate impactx-openmp-dev

again.
Replace ``openmp`` with the equivalent you chose.

Compile ImpactX with ``-DImpactX_MPI=ON``.
For ``pip``, use ``export IMPACTX_MPI=ON``.


Brew (macOS/Linux)
------------------

`Homebrew (Brew) <https://brew.sh>`__ is a user-level package manager primarily for `Apple macOS <https://en.wikipedia.org/wiki/MacOS>`__, but also supports Linux.

.. code-block:: bash

   brew update
   brew tap openpmd/openpmd
   brew install adios2      # for openPMD
   brew install ccache
   brew install cmake
   brew install fftw        # for IGF, CSR
   brew install git
   brew install hdf5-mpi    # for openPMD
   brew install libomp
   brew unlink gcc
   brew link --force libomp
   brew install pkg-config  # for fftw
   brew install open-mpi
   brew install openblas    # for PSATD in RZ
   brew install openpmd-api # for openPMD

If you also want to compile with PSATD in RZ, you need to manually install BLAS++ and LAPACK++:

.. code-block:: bash

   sudo mkdir -p /usr/local/bin/
   sudo curl -L -o /usr/local/bin/cmake-easyinstall https://raw.githubusercontent.com/ax3l/cmake-easyinstall/main/cmake-easyinstall
   sudo chmod a+x /usr/local/bin/cmake-easyinstall

   cmake-easyinstall --prefix=/usr/local git+https://github.com/icl-utk-edu/blaspp.git \
       -Duse_openmp=OFF -Dbuild_tests=OFF -DCMAKE_VERBOSE_MAKEFILE=ON
   cmake-easyinstall --prefix=/usr/local git+https://github.com/icl-utk-edu/lapackpp.git \
       -Duse_cmake_find_lapack=ON -Dbuild_tests=OFF -DCMAKE_VERBOSE_MAKEFILE=ON

Compile ImpactX with ``-DImpactX_MPI=ON``.
For ``pip``, use ``export IMPACTX_MPI=ON``.


APT (Debian/Ubuntu Linux)
-------------------------

The `Advanced Package Tool (APT) <https://en.wikipedia.org/wiki/APT_(software)>`__ is a system-level package manager on Debian-based Linux distributions, including Ubuntu.

.. tab-set::

   .. tab-item:: With MPI (only Linux/macOS)

      .. code-block:: bash

         sudo apt update
         sudo apt install build-essential ccache cmake g++ git libfftw3-mpi-dev libfftw3-dev libhdf5-openmpi-dev libopenmpi-dev pkg-config python3 python3-matplotlib python3-numpy python3-pandas python3-pip python3-scipy python3-venv

         # optional:
         # for CUDA, either install
         #   https://developer.nvidia.com/cuda-downloads (preferred)
         # or, if your Debian/Ubuntu is new enough, use the packages
         #   sudo apt install nvidia-cuda-dev libcub-dev

         # compile ImpactX with -DImpactX_MPI=ON
         # for pip, use: export IMPACTX_MPI=ON

   .. tab-item:: Without MPI

      .. code-block:: bash

         sudo apt update
         sudo apt install build-essential ccache cmake g++ git libfftw3-dev libfftw3-dev libhdf5-dev pkg-config python3 python3-matplotlib python3-numpy python3-pandas python3-pip python3-scipy python3-venv

         # optional:
         # for CUDA, either install
         #   https://developer.nvidia.com/cuda-downloads (preferred)
         # or, if your Debian/Ubuntu is new enough, use the packages
         #   sudo apt install nvidia-cuda-dev libcub-dev

         # compile ImpactX with -DImpactX_MPI=OFF
         # for pip, use: export IMPACTX_MPI=OFF
