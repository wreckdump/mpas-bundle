# MPAS-Bundle AOCC Support

This repository is a fork of the JCSDA MPAS-Bundle repository with additional
modifications required to build MPAS-JEDI using the AMD AOCC compiler stack.

The primary goal of this fork is to provide a reproducible build environment
for AMD systems using AOCC compilers, particularly AOCC Flang.

Original repository:
https://github.com/JCSDA/mpas-bundle

Fork:
https://github.com/wreckdump/mpas-bundle


## Motivation

The upstream JCSDA repositories are actively developed and periodically update
their internal dependencies. During testing with AOCC 5.2, several issues were
identified that prevent a clean build using the default upstream development
branches.

The required changes are mainly compatibility fixes for the following JCSDA
components:

- ioda
- oops
- saber
- ufo
- vader

The modifications are maintained in separate AOCC-specific branches in this
fork.


## AOCC Support Branches

The following repositories use the `aocc-support` branch:

- oops
  https://github.com/wreckdump/oops/tree/aocc-support

- vader
  https://github.com/wreckdump/vader/tree/aocc-support

- saber
  https://github.com/wreckdump/saber/tree/aocc-support

- ioda
  https://github.com/wreckdump/ioda/tree/aocc-support

- ufo
  https://github.com/wreckdump/ufo/tree/aocc-support


Other dependencies continue to use the upstream JCSDA repositories.

Currently unchanged:

- CRTM
  https://github.com/JCSDA/CRTMv3

- mpas-jedi
  https://github.com/JCSDA/mpas-jedi


## Changes in This Fork

The top-level CMake configuration has been modified so that the AOCC build
uses the AOCC-compatible branches.

The following entries were changed from:

    https://github.com/JCSDA/<repository>.git
    BRANCH develop

to:

    https://github.com/wreckdump/<repository>.git
    BRANCH aocc-support


Affected repositories:

- oops
- vader
- saber
- ioda
- ufo


## Building With AOCC

Example configuration:

    mkdir build
    cd build

    cmake .. \
      -DCMAKE_C_COMPILER=clang \
      -DCMAKE_CXX_COMPILER=clang++ \
      -DCMAKE_Fortran_COMPILER=mpifort \
      -DCMAKE_C_FLAGS="-Wno-error -march=znver2" \
      -DCMAKE_CXX_FLAGS="-stdlib=libc++ -Wno-error -Dgsl_FEATURE_GSL_COMPATIBILITY_MODE=1 -Dgsl_CONFIG_DEFAULTS_VERSION=1 -march=znver2" \
      -DCMAKE_Fortran_FLAGS="-Wno-error -march=znver2 -ffree-form -DSINGLE_PRECISION" \
      -DCMAKE_Fortran_FLAGS_RELEASE="-O3 -g -march=znver2 -ffree-form -DSINGLE_PRECISION" \
      -DNetCDF_CXX_LIBRARY=/usr/lib/libnetcdf-cxx4.so \
      -DNetCDF_CXX_INCLUDE_DIR=/usr/include \
      -DOOPS_ENABLE_STACKTRACE=OFF \
      -DOOPS_STACKTRACE_PROVIDER=none \
      -DBUILD_TESTING=OFF \
      -DENABLE_TESTS=OFF \
      -DCMAKE_BUILD_TYPE=Release


Then:

    make -j


## Known AOCC Issue Fixed

### SABER compilation failure

The upstream SABER development branch introduced changes in
SaberCentralBlock that are incompatible with the previous dependency state
used by MPAS-Bundle.

The failure appears as:

    error: use of undeclared identifier 'groupOuterBlockChains_'

and:

    error: no member named 'outerBlocks' in
    'saber::SaberCentralBlockGroupParameters'


The AOCC-support branch retains the compatible SABER implementation required
for this build environment.


## Updating Upstream Dependencies

The upstream JCSDA repositories are under active development.

To update:

1. Fetch upstream changes:

       git fetch upstream

2. Review changes:

       git log upstream/develop

3. Update individual AOCC branches as needed.

Example:

       cd saber

       git fetch upstream

       git checkout aocc-support

       git merge upstream/develop


Changes should be tested with the AOCC build before merging.


## Relationship With Upstream

This fork is not intended to replace the official JCSDA repositories.

The purpose is to provide:

- AOCC compiler compatibility
- reproducible builds
- a place to maintain compiler-specific patches

If the fixes become generally applicable, they should eventually be submitted
upstream as pull requests to the corresponding JCSDA repositories.


## Environment

Tested with:

- AMD AOCC 5.2
- LLVM/Flang based Fortran compiler
- AMD Zen architecture
- MPAS-JEDI development environment


## License

This repository follows the same license as the original JCSDA MPAS-Bundle
project.
