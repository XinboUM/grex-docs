---
title: Code Development on Grex
bookCollapseSection: true
weight: 70
---

# Code Developing on Grex

Grex comes with a sizeable software stack that contains most of the software development environment for typical HPC applications. This section of the documentation covers the best practices for compiling and building your own software on Grex.

The login nodes of Grex can be used to compile code and to run short interactive and/or test runs. All other jobs must be submitted to the batch system. We do not do as heavy resource limiting on Grex login nodes as, for example, Compute Canada does; so code development on login nodes is entirely possible. However, it might still make sense to perform some of the code development in interactive jobs, in cases of a) the build process and/or tests requires heavy, many-core computations and/or b) you need access to specific hardware not present on the login nodes, such as GPUs and AVX512 CPUs.

Most of the software on Grex is available through environmental modules. To find a software development tool or a library to build your code against, the _module spider_ command is a good start. The applications software is usually installed by us from sources, into sub-directories under _/global/software/cent7_

It is almost always better to use communication libraries (MPI) provided on Grex rather than building your own, because ensuring tight
integration of these libraries with our SLURM scheduler and low-level, interconnect-specific libraries might be tricky.

## General CentOS-7 notes

The base operating system on Grex is CentOS 7.x that comes with its set of development tools. However, due to the philosophy of CentOS, the tools are usually rather old. For example, _cmake_ and _git_ are of ancient versions of 2.8, 1.7 correspondingly. Therefore, even for these basic tools you more likely want to load a module with newest verstions of these tools:

```module load git```

```module load cmake```

CentOS also has its system versions of Python, Perl, and GCC compilers. When no modules loaded, the binaries of these will be available in the PATH. The purpose of these is to make some systems scripts possible, to compile OS packages, drivers and so on.

We do not install many packages for the dynamic languages system-wide, because it makes maintaining different versions of them complicated. The same adivce applies: use the **module spider** command to find a version of Perl, Python, R, etc. to suit you. The same applies to compiler suites like GCC and Intel.

We do install CentOS packages with OS that are a) base OS things necessary for functioning, b) graphical libraries that have many dependencies, c) never change versions that are not critical for performance and/or security. Here are some examples: FLTK, libjpeg, PCRE, Qt4 and Gtk. Login nodes of Grex have many ''-devel'' packages installed, while compute nodes do not because we want them lean and quickly reinstallable. Therefore, compiling codes that requires ''-devel'' base OS packages might fail on compute nodes. Contact us us if something like taht happens when compiling or running your applications.

Finally, because HPC machines are shared systems and users do not have ''sudo'' access, following some instructions from a Web page that asks for ''apt-get install this '' or ''yum install that'' will fail. Rather, _module spider_ should be used to see if the package you want is already installed and available as a module.

## Compilers and Toolchains

Due to the hierarchical nature of our **Lmod** modules system, compilers and certain core libraries (MPI and CUDA) form toolchains. Normally, you would need to chose a compiler suite (Intel or GCC) and, in case of parallel applications, a MPI library (OpenMPI or IntelMPI). These come in different versions. Also, you'd want to know if you want CUDA should your applications be able to utilize GPUs. A combination of compiler/version, MPI/version and possibly CUDA makes a toolchain. Toolchains are mutually exclusive; you cannot mix software items compiled with different toolchains!

See Modules for more information.

_There is no modules loaded by default_ ! There will be only system's GCC-4.8 and no MPI whatsoever. To get started, load a compiler/version. Then, if necessary, an MPI (ompi or impi) and if necessary, CUDA (for which 10.2 is the current version, there is no known reason to use another).

```module load intel/15.0```

```module load ompi/3.1.4```

The above loads Intel compilers and OpenMPI 3.1.4.

```module load gcc/5.2```

```module load ompi/3.1.4```

The MPI wrappers (_mpicc_, _mpicxx_, _mpif90_, ... etc.)  will be set correctly by _ompi_ modules to point to the right compiler.

### Intel compilers suite

At the moment of writing this documentation, the following Intel Compilers Suites are available on Grex:

  - Intel 15.0
  - Intel 14.0
  - Intel 12.1

The name for the Intel suite modules is _intel_; ```module spider intel``` is the command to find available Intel versions. The later is left for compatibility with legacy codes. It does not work with systems C++ standard libraries well, so _icpc_ for Intel 12.1 might be dysfunctional. So the intel/12.1 toolchain is actually using GCC 4.8's C++ and C compilers. 

If unsure, or do not have a special reason otherwise, use Intel 15.0 compilers (icc, icpc, ifort). Intel 15.0 is probably the first Intel compiler to support AVX512 if you are going to use the contributed nodes that have AVX512 architecture.

The Intel compilers suite also provides tools and libraries such as [MKL](https://software.intel.com/en-us/mkl) (Linear Algebra, FFT, etc.), Intel Performance Primitives (IPP), Intel [Threads Building Blocks](https://software.intel.com/en-us/tbb) (TBB), and [VTune](https://software.intel.com/en-us/vtune). Intel MPI as well as MKL for GCC compilers are available as separate modules, should they be needed for use separately.

### GCC compilers suite

At the moment of writing this page, the following GCC compilers are available:

 - GCC 9.2
 - GCC 7.4
 - GCC 5.2
 - GCC 4.8

The name for GCC is _gcc_, as in ```module spider gcc```. The GCC 4.8 is a place holder module; its use is to unload any other modules of the compiler family, to avoid toolchains conflicts. Also, GCC 4.8 is the only multilib GCC compiler around; all the others are strictly **64-bit**, and thus unable to compile legacy **32-bit** programs.

For utilizing of the AVX512 instructions, probably the best way is to go with the latest GCC compilers (9.2 and 7.4) and latest MKL. GCC 4.8 does not handle AVX512. Generally Intel compilers outperform GCC, but GCC might have better support for the recent C++11,14,17 standards.

## MPI and Interconnect libraries

The standard distribution of MPI on Grex is [OpenMPI](https://www.open-mpi.org/). We build most of the software with it. To keep compatibility with the old Grex software stack, we name the modules _ompi_. MPI modules depend on the compiler they were built with, which means, that a compiler module should be loaded first; then the dependent MPI modules will become available as well. Changing the compiler module will trigger automatic MPI module reload. This is how Lmod hierarchy works now.

For a long time Grex was using the interconnect drivers with ibverbs packages from the IB hardware vendor, Mellanox. It is no longer the case: for CentOS-7, we have switched to the vanilla Linux infiniband drivers, the open source RDMA-core package, and OpenUCX libraries. The current version of UCX on Grex is 1.6.1. Recent versions of OpenMPI (3.1.x and 4.0.x) do support UCX. Also, our OpenMPI is built with process management interface versions PMI1, PMIx2 and 3, for tight integration with the SLURM scheduler.

The current default and recommended version of MPI is OpenMPI 3.1.4. There is a newer version, OpenMPI 4.0.2 that works well for then new codes, but might break compatibility with older MPI codes because it is a major old-code cleanup version. Therefore, most of Grex  users might want to go with this:

```module load ompi/3.1.4```

There is also IntelMPI, for which the modules are named _impi_. See the notes on running MPI applications under SLURM [here](/doc/docs/grex/running/batch).

All MPI modules, be that OpenMPI or Intel, will set MPI compiler wrappers such as _mpicc_, _mpicxx_, _mpif90_ to the compiler suite they were built with. The typical workflow for building parallel programs with MPI would be to first load a compiler module, then an MPI module, and then use the wrapper of C, C++ or Fortran in your makefile or build script.

In case a build or configure script does not want to use the wrapper and needs explicit compiler and link options for MPI, OpenMPI wrappers provide the _--show_ option that list the required command line options. Try for example:

```mpicc --show```

To print include and library flags to the C compiler to be linked against currently loaded OpenMPI. 

## Linear Algebra BLAS/LAPACK

It is always a bad idea to use the reference BLAS/LAPACK/CLAPACK from Netlib (or the generic -lblas, -llapack from CentOS or EPEL which also likely is the reference BLAS/LAPACK from Netlib). The physical Computer architecture has much evolved, and is now way different from the logical Computer the human programmer is presented with. Todays, it takes careful, manual assembly coding optimization to implement BLAS/LAPACK that performs fast on modern CPUs with their memory hierarchies, instruction prefetching and speculative execution. A vendor-optimized BLAS/LAPACK implementation should always be used. For the Intel/AMD architectures, it is Intel MKL, OpenBLAS, and BLIS.

ALso, it is worth noting that the linear algebra libraries might come with two versions: one 32 bit array indexes, another of full 64 bit. Users must pay attention and link against the proper version for their software (that is, a Fortran codes with -i8 or -fdefault-integer-8 would link against 64 bit pointers BLAS).

### MKL

The fastest BLAS/LAPACK implementation from Intel. With Intel compilers, it can be used as a convenient compiler flag, _-mkl_ or if threaded version is not needed, _-mkl=sequential_.

With both Intel and GCC compilers, the MKL libraries can be linked explicitly with compiler/linker options. The base path for MKL includes and libraries is defined as the _MKLROOT_ environment variable. For GCC compilers, ```module load mkl``` is needed to add _MKLROOT_ to the environment. There is a command line [advisor](https://software.intel.com/en-us/articles/intel-mkl-link-line-advisor) Website to pick the correct order and libraries. Libraries with the \_ilp64 suffix are for 64 bit indexes while \_lp64 are for the default, 32 bit indexes.

Note that when Threaded MKL is used, the number of threads is controlled with _MKL_NUM_THREADS_ environment variable. On Grex software stack, it is set by the MKL module to 1 to prevent accidental CPU over-subscription. Redefine it in your SLURM job scripts if you really need threaded MKL execution as follows:

```export MKL_NUM_THREADS=$SLURM_CPUS_PER_TASK```

We use the MKL's BLAS and LAPACK for compiling R and Python's NumPY package on Grex, and thats one example when threaded MKL can speed up computations if the code spends significant time in linear algebra routines by using SMP.

Note that MKL also provides ScaLAPACK and FFTW libraries.

### OpenBLAS

The successor and continuation of the famous GotoBLAS2 library. It contains both BLAS and LAPACK in a sigle library, _libopenblas.a_ . Use ''module spider openblas'' to find available versions for a given compiler suite. We provide both 32 bit and 64 bit indexes versions (and reflect ot in the version names, like _openblas/0.3.7-i32_). The [performance of OpenBLAS](https://software.intel.com/en-us/articles/performance-comparison-of-openblas-and-intel-math-kernel-library-in-r) is close to that of MKL.

### BLIS

[Blis](https://en.wikipedia.org/wiki/BLIS_(software)) is a recent, C++ template based implementation of linear algebra that contains a BLAS interface. On Grex, only 32 bit indexes BLIS is available at the moment. Use ''module spider blis'' to see how to load it.

### ScaLAPACK

MKL has ScaLAPACK included. Note that it depends on BLACS which in turn depends on an MPI version. MKL comes with support of OpenMPI and IntelMPI for the BLACS layer; it is necessary to pick the right library of them to link against.

The [command line advisor](https://software.intel.com/en-us/articles/intel-mkl-link-line-advisor) is helpful for that.

## Fourier Transoform FFTW

FFTW3 is the standard and well performing implementation of FFT. ```module spider fftw``` should find it. There is parallel version of the FFTW3 that depends on MPI it uses, thus to load the _fftw_ module, compiler and MPI modules would have to be loaded first. MKL also provides FFTW bindings, which can be used as follows:

Either of Intel or GCC MKL modules would set the _MKLROOT_ environment variable, and add necessary directories to _LD_LIBRARY_PATH_. The _MKLROOT_ is handy when using explicit linking against libraries. It can be useful if you want to select a particular compiler (Intel or GCC), pointer width (the corresponding libraries have suffix _lp64 for 32 bit pointers and_ilp64 for 64 bit ones; the later is needed for, for example, Fortran codes with INTEGER*8 array indexes, explicit or set by -i8 compiler option) and kind of MPI library to be used in BLACS (OpenMPI or IntelMPI which both are available on Grex). An example of the linker options to link against sequential, 64 bit pointed vesion of BLAS, LAPACK for an Intel Fortran code is:

```ifort -O2 -i8 main.f -L$MKLROOT/lib/intel64 -lmkl_intel_ilp64 -lmkl_sequential -lmkl_core -lpthread -lm```

MKL also has FFTW bindings. They have to be enabled separately from the general Intel compilers installation; and therefore details of the usage might be different between different clusters. On Grex, these libraries are pesent in two versions: 32 bit pointers (libfftw3xf_intel_lp64) and 64 bit pointers (fftw3xf_intel_ilp64). To link against these FFT libraries, the following include and library options to the compilers can be used (for the _lp64 case):

```-I$MKLROOT/include/fftw -I$MKLROOT/interfaces/fftw3xf -L$MKLROOT/interfaces/fftw3xf -lfftw3xf_intel_lp64```

The above line is, admittedly, rather elaborate but give the benefit of compiling and building all of the code with MKL, without the need for maintaining a separtate library such as FFTW3.

## HDF5 and NetCDF

Popular hierarchical data formats. Two versions exist on the Grex software stack, one serial and another MPI-dependent version. Which one you load depends whether MPI is loaded.
"
```module spider hdf5```

```module spider netcdf```

## Python

There are Conda python modules and the Python built from sources with variety of the compilers. The conda based modules can be distinguished by the module name. Note that the base OS python should in most cases not be used; rather use a module:

```module spider python```

We do install certain most popular python modules (such as Numpy, Scipy, matplotlib) centrally. _pip list_ and _conda list_ would show the instaled modules 

## R

We build R from sources and link against MKL. There are Intel 15 versions of R; however we find that some packages would only work with GCC-compiled versions of R because they assume GCC or rely on some C++11 features that the older Intel C++ might be lacking. To find available modules for R, use:

```module spider "r"```

Several R packages are installed with the R modules on Grex. Note that it is often the case that R packages ar bindings for some other software (JAGS, GEOS, etc.) and require the software or its dynamic libraries to be available at runtime. This means, the modules for the dependencies (JAGS, GEOS) are also to be loaded when R is loaded.

