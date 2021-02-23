*toolbox* library and utilities
===============================

A set of utilities and library functions to facilitate writing of 
scientific codes -- with a particular focus on the analysis of 
atomistic simulations.

Compilation and dependencies
----------------------------

Compilation of the *toolbox* libraries and tools should not be too
difficult. It requires that FFTW3 and LAPACK libraries are installed
on the system, and a C++ compiler. 

On a Ubuntu system the dependencies can be satisfied by installing

```bash
sudo apt-get install build-essential liblapack-dev libfftw3-dev
```
Then, compiling *toolbox* should be straightforward:

```bash
cd src
make
```

The resulting executables will be generated in `bin`.

One can modify the default parameters for the compiler and library 
paths by creating a arch.xxx file in the main path, and specifying
which architecture file to load by running

```bash
cd src
ARCH=xxx make
```


## CNQ
```
sudo apt-get install libboost-all-dev
```
自己编译的fftw，` vi tools/Makefile`
```
CXXFLAGS:=$(CXXFLAGS) -I../libs -I/home/cndaqiang/code/fftw-3.3.4/include
```
编译boost
下载,https://sourceforge.net/projects/boost/files/boost/
```

```

