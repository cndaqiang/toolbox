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

编译过程

### gcc7/8
### fftw
```
ROOT=$PWD
mkdir $ROOT/source
#fft
cd $ROOT/source
wget https://cndaqiang.gitee.io/packages//mirrors/fftw/fftw-3.3.3.tar.gz
tar xzvf fftw-3.3.3.tar.gz
cd fftw-3.3.3/
./configure --prefix=$ROOT/fftw-3.3.3 # --enable-mpi --enable-shared
make install -j20
```
### lapack
```

#从0编译动态库scalapack
if [ ! -d $ROOT/math ]
then
    mkdir $ROOT/math
    mkdir $ROOT/math/lib
    mkdir $ROOT/math/include
fi

cd $ROOT/source
wget https://cndaqiang.gitee.io/packages//mirrors/math/lapack-3.8.0.tar.gz
rm -rf lapack-3.8.0
tar xzvf lapack-3.8.0.tar.gz
cd lapack-3.8.0/
#修改编译参数，并默认输出动态库

#BLAS&LAPACK
echo -e "
all:libblas.so
libblas.so: \$(ALLOBJ)
\t \$(FORTRAN) -shared -Wl,-soname,\$@ -o \$@ \$(ALLOBJ)
" >> BLAS/SRC/Makefile
#TMG
echo -e "
all:libtmg.so
libtmg.so: \$(ALLOBJ)
\t \$(FORTRAN) -shared -Wl,-soname,\$@ -o \$@ \$(ALLOBJ)
" >> TESTING/MATGEN/Makefile
#LAPACK
echo -e "
all: liblapack.so
liblapack.so: \$(ALLOBJ)
\t \$(FORTRAN) -shared -Wl,-soname,\$@ -o \$@ \$(ALLOBJ)
" >> SRC/Makefile

cp make.inc.example make.inc
echo "OPTS += -fPIC" >> make.inc
echo "NOOPT += -fPIC" >> make.inc
echo -e "
MATHDIR=$ROOT/math/lib
install:all
\t cp $PWD/BLAS/SRC/libblas.so \$(MATHDIR)
\t cp $PWD/SRC/liblapack.so \$(MATHDIR)
\t cp $PWD/TESTING/MATGEN/libtmg.so \$(MATHDIR)
\t cp $PWD/*.a \$(MATHDIR)
" >> make.inc
#安装
make -j36 #必须用make，用其他参数/完成后再输make会编译测试程序，没必要
```
### boost
```
cd $ROOT/source
wget https://cndaqiang.gitee.io/packages//mirrors/boost/boost_1_66_0.tar.bz2
tar --bzip2 -xf boost_1_66_0.tar.bz2
cd boost_1_66_0/
./bootstrap.sh --prefix=$ROOT/boost_1_66_0
./b2 install
#编译过程很多faild, 不用管
```
### toolbox
#### 环境参数
```
echo "
ROOT=$PWD
export LD_LIBRARY_PATH=${ROOT}/math/lib:$LD_LIBRARY_PATH
export LIBRARY_PATH=${ROOT}/math/lib:$LIBRARY_PATH
export PATH=$PATH:/public/home/chendq/soft/gnu8/toolbox/bin
ROOT=/public/home/chendq/soft/gnu8
export LD_LIBRARY_PATH=${ROOT}/math/lib:$LD_LIBRARY_PATH
export LIBRARY_PATH=${ROOT}/math/lib:$LIBRARY_PATH
export C_INCLUDE_PATH=${ROOT}/math/include:$C_INCLUDE_PATH
export CPLUS_INCLUDE_PATH=${ROOT}/math/include:$CPLUS_INCLUDE_PATH
export PATH=${ROOT}/math/bin:$PATH
export LD_LIBRARY_PATH=${ROOT}/fftw-3.3.3/lib:$LD_LIBRARY_PATH
export LIBRARY_PATH=${ROOT}/fftw-3.3.3/lib:$LIBRARY_PATH
export C_INCLUDE_PATH=${ROOT}/fftw-3.3.3/include:$C_INCLUDE_PATH
export CPLUS_INCLUDE_PATH=${ROOT}/fftw-3.3.3/include:$CPLUS_INCLUDE_PATH
export PATH=${ROOT}/fftw-3.3.3/bin:$PATH
export LD_LIBRARY_PATH=${ROOT}/boost_1_66_0/lib:$LD_LIBRARY_PATH
export LIBRARY_PATH=${ROOT}/boost_1_66_0/lib:$LIBRARY_PATH
export C_INCLUDE_PATH=${ROOT}/boost_1_66_0/include:$C_INCLUDE_PATH
export CPLUS_INCLUDE_PATH=${ROOT}/boost_1_66_0/include:$CPLUS_INCLUDE_PATH
export PATH=${ROOT}/boost_1_66_0/bin:$PATH
" > $ROOT/toolbox.sh
source $ROOT/toolbox.sh
echo alias toolbox=source $ROOT/toolbox.sh >> ~/.bashrc
```
#### 正式编译
```
cd $ROOT
git clone https://github.com/cndaqiang/toolbox.git

echo CXXFLAGS += -I/home/apps/mathlib/fftw/gnu8/3.3.3/include -I/home/apps/mathlib/boost/gcc8.4/boost_1_66_0/include >> $ROOT/toolbox/make.in
cd toolbox/src
make
```

#### 之前的草稿
```
添加fftw和boost到头文件
```
prepend-path    CPLUS_INCLUDE_PATH      ${fftw_HOME}/include
prepend-path    CPLUS_INCLUDE_PATH      ${boost_HOME}/include
```
或者填到`toolbox/make.in`

```
CXXFLAGS=-O3 -ftracer -g  -DTB_FFTAC -D__USEREGEX=1 -I/home/apps/mathlib/fftw/gnu8/3.3.3/include -I/home/apps/mathlib/boost/gcc8.4/boost_1_66_0/include
```
最终toolbox运行的module file
```
#%Module

prepend-path    PATH                    /home/apps/toolbox/gcc8.4/toolbox/bin/
#env
set SOFT_HOME   /home/apps/gcc8.4

prepend-path    LIBRARY_PATH            ${SOFT_HOME}/lib
prepend-path    LIBRARY_PATH            ${SOFT_HOME}/lib64

prepend-path    LD_LIBRARY_PATH            ${SOFT_HOME}/lib
prepend-path    LD_LIBRARY_PATH            ${SOFT_HOME}/lib64

prepend-path    PATH                    ${SOFT_HOME}/bin
prepend-path    CPLUS_INCLUDE_PATH      ${SOFT_HOME}/include


set boost_HOME /home/apps/mathlib/boost/gcc8.4/boost_1_66_0
prepend-path    LIBRARY_PATH            ${boost_HOME}/lib
prepend-path    LD_LIBRARY_PATH         ${boost_HOME}/lib
prepend-path    INCLUDE                 ${boost_HOME}/include
prepend-path    C_INCLUDE_PATH          ${boost_HOME}/include
prepend-path    CPLUS_INCLUDE_PATH      ${boost_HOME}/include

set fftw_HOME /home/apps/mathlib/fftw/gnu8/3.3.3
prepend-path    LIBRARY_PATH            ${fftw_HOME}/lib
prepend-path    LD_LIBRARY_PATH         ${fftw_HOME}/lib
prepend-path    INCLUDE                 ${fftw_HOME}/include
prepend-path    C_INCLUDE_PATH          ${fftw_HOME}/include
prepend-path    PATH                    ${fftw_HOME}/bin
prepend-path    CPLUS_INCLUDE_PATH      ${fftw_HOME}/include

set SCALAPACK_HOME  /home/apps/mathlib/scalapack/gcc8.4/2.1.0
prepend-path    LIBRARY_PATH            ${SCALAPACK_HOME}/lib
prepend-path    INCLUDE                 ${SCALAPACK_HOME}/include
prepend-path    LD_LIBRARY_PATH         ${SCALAPACK_HOME}/lib
```

