---
title: LAMMPS and MLIP on Mac
author: Carlos Leon
date: 2021-04-10
categories: [Blogging, LAMMPS, MLIP]
tags: [LAMMPS, MTP, MLIP, install, Mac]
<!-- pin: true -->
---

## Install OpenMPI

First, install OpenMPI following directions [here](http://www.science.smith.edu/dftwiki/index.php/Install_MPI_on_a_MacBook)
```bash
cd openmpi*
./configure
make all install
```
OpenMPI creates the following files on /usr/local/bin . Check it is in your PATH
```bash
❯ ll mpi*
Permissions Size User     Date Modified Date Created Date Accessed Name
lrwxr-xr-x    12 chinchay  9 Apr 23:15   9 Apr 23:15  9 Apr 23:15  mpic++ -> opal_wrapper
lrwxr-xr-x    12 chinchay  9 Apr 23:15   9 Apr 23:15  9 Apr 23:15  mpicc -> opal_wrapper
lrwxr-xr-x    12 chinchay  9 Apr 23:15   9 Apr 23:15  9 Apr 23:15  mpicxx -> opal_wrapper
lrwxr-xr-x     7 chinchay  9 Apr 23:14   9 Apr 23:14  9 Apr 23:14  mpiexec -> orterun
lrwxr-xr-x    12 chinchay  9 Apr 23:15   9 Apr 23:15  9 Apr 23:15  mpif77 -> opal_wrapper
lrwxr-xr-x    12 chinchay  9 Apr 23:15   9 Apr 23:15  9 Apr 23:15  mpif90 -> opal_wrapper
lrwxr-xr-x    12 chinchay  9 Apr 23:15   9 Apr 23:15  9 Apr 23:15  mpifort -> opal_wrapper
lrwxr-xr-x     7 chinchay  9 Apr 23:14   9 Apr 23:14  9 Apr 23:14  mpirun -> orterun
❯ pwd
/usr/local/bin
```

## LAMMPS as shared library

First, use `virtualenv` (in Graham) or `pyenv` (in my local machine) to define the python version:
```bash
pyenv local <myenv>
```


Download LAMMSP:

```bash
git clone https://github.com/lammps/lammps.git
cd lammps/src/MAKE/OPTIONS
```
I tried with other options (Makefile.mpi, Makefile.serial, Makefile.mac) but they didn't work, MacOS Big Sur complains about not finding compatibility with `c++11`. `Makefile.g++_openmpi` worked and it already contains the flag `mpicxx -std=c++11`.

```bash
cd ../../ # return to lammps/src
make yes-molecule
make g++_openmpi mode=shlib
```
At this point, `*.so` libraries have been created in `lammps/src`.


## MLIP

Download MLIP and `cd` into the directory
```bash
export mylammps="<path/to/LAMMPS>"
./configure --lammps=$mylammps
```
For Big Sur, I needed to modify configuration files:

* MLIP will be unable to find the gfortran library (errors with -lgfortran). Tell it manually where to search for the library through the flag `-L` (while `-l` is used to make reference to the library itself). I ended up installing gfortran by downloading it, instead of using it using `brew`, so in my case, I need to check for the following lines in `make/config.mk`:

```bash
LDFLAGS += $(LIB_DIR)/lib_mlip_cblas.a -L/usr/local/gfortran/lib -lgfortran
...
LAMMPS_SYSLIB =  $(abspath $(LIB_DIR)/lib_mlip_cblas.a) -L/usr/local/gfortran/lib -lgfortran
```

If using gfortran from brew, check `libgfortran.a` is in `/usr/local/Cellar/gcc/10.2.0_4/lib/gcc/10` (or modifiy it according to your gcc version), and add it. Example: `-L/usr/local/Cellar/gcc/10.2.0_4/lib/gcc/10`


* In a later step installation, LAMMPS will complain again about compatibiligy issues with c++11, so add this flag in `make/config.mk`:
```bash
CXX_EXE = mpicxx -std=c++11
```

* Also, modify the `make/config.mk` to the type of machine you chose in LAMMPS installation:
```bash
LAMMPS_TARGET_MPI = g++_openmpi
```

* The default value of `LAMMPS_TARGET_SERIAL = serial` in `make/config.mk` will look  for `Makefile.serial` in `<path/to/LAMMPS>/src/MAKE` and errors about `std=c++11` will be found if not adding the following to the Makefile:
```bash
CC =  g++  -std=c++11
```

Now you are ready to install MLIP, and use it through LAMMPS:
```bash
make clean
make mlp
make lammps-shlib
```

Finally, save the paths into `.bash_profile` or `.zshrc`:

```bash
export mylammps="<path/to/LAMMPS>"
export mymlip="<path/to/MLIP>" # not the bin/ directory, but the parent one, needed to find lib/
export PYTHONPATH=$mylammps/python:$PYTHONPATH
export LD_LIBRARY_PATH=$mylammps/src:/mymlip/lib:$LD_LIBRARY_PATH ## will search for $mylammps/src/liblammps_intel_cpu_intelmpi.so and /mlip/lib/liblammps_mpi.so
```


In python:

```python
from lammps import lammps
lmp = lammps()
```
Change the extension of `liblammps.so` to `liblammps.dylib` if python can't find the library. `<path/to/LAMMPS>/python/lammps/core.py` will look for the extension `*.dylib` for Darwin (Mac) instead of `*.so` for Linux.
