---
layout: post
title: Install HEALPix and PolSpice in a conda environment
categories: [conda,healpy,nersc]
---

Some notes on how to install HEALPix and PolSpice inside a conda environment,
with some details about doing it at NERSC, but most of the tutorial is independent of that.

## Setup the conda environment and the compilers

I assume here we are installing it into a custom conda environement
the possibly contains all other cosmology packages, like `healpy`.

For example created with:

    module load python
    conda create -n pycmb python==3.7 healpy matplotlib ipykernel

When you activate the conda environment, the variable `$CONDA_PREFIX` is
automatically set to the base folder of the environment,
something like:

    ~/anaconda/envs/pycmb

To make it simpler, I am using `gcc` and `gfortran`, if at NERSC run:

    module load PrgEnv-gnu

if that fails, probably you need first to unload the Intel environment:

    module unload PrgEnv-intel
    module load PrgEnv-gnu

## Install cfitsio

`cfitsio` is quite easy, better download the version included in
`healpy` because it has a couple of fixes:

    git clone https://github.com/healpy/cfitsio
    cd cfitsio

We want to install it into a dedicated folder, not the same `lib` folder
of the conda environment, so that we don't risk to have conflicts
with the compiler libraries during the build process:

    ./configure --prefix=$CONDA_PREFIX/cfitsio
    make -j8 shared install

## Install HEALPix

HEALPix installs itself in the same folder where it is unpacked, and then modifies the bash
profile to make things work.
As we want to keep things isolated, let's unpack the Healpix package into the conda environment folder, so it will be something like:

    $CONDA_PREFIX/Healpix_3.70

    ./configure

configure the C, the Fortran packages, the Fortran package requires `libsharp`,
set everything to default except location of `cfitsio` where you need (notice `lib` at the end):

    $CONDA_PREFIX/cfitsio/lib

When the installer asks whether to modify `.profile` respond no.
Now the installer will create some scripts in the `~/.healpix` folder and modify `.profile`, we want to only activate HEALPix in our conda environment so we should modify `.profile` and remove the lines added by HEALPix.

Finally we can have HEALPix automatically activated when the conda environment is initialized (notice we need the script to end in `.sh`):

    mkdir -p ${CONDA_PREFIX}/etc/conda/activate.d
    ln -s ~/.healpix/3_70_Linux/config ${CONDA_PREFIX}/etc/conda/activate.d/config.sh

Restart the conda environment, and try to run `anafast` to check that it works.
If you are at NERSC, make sure that you are always loading the GNU programming environment by having:

    module swap PrgEnv-intel PrgEnv-gnu

in `.bashrc.ext`.

## Install PolSpice

Create a `build` folder inside the source folder and create a `run.sh` file with this content:

    cmake .. -DCFITSIO=${CONDA_PREFIX}/cfitsio/lib -DCMAKE_Fortran_COMPILER=gfortran -DCMAKE_C_COMPILER=gcc

Then:

    bash run.sh
    make -j8

This will put the `spice` executable into the `../bin` folder, just copy it to the conda environment bin folder:

    cd ..
    ln -s $(pwd)/bin/spice ${CONDA_PREFIX}/bin/

We can also copy the 2 python modules into the environment:

    ln -s $(pwd)/bin_llcl.py $(pwd)/ispice.py ${CONDA_PREFIX}/lib/python3.*/site-packages/

Check it works:

    spice -usage
