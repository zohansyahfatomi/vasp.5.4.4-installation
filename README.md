# vasp.5.4.4-installation

## Requirements (in my case):
- Ubuntu 20.04 LTS (focal)
- VASP 5.4.4

## Install GNU Compiler
```
sudo apt-get install make build-essential g++ gfortran
```

## Install required VASP libraries (lapack, scalapack, openmpi, and fftw)
```
sudo apt-get install libblas-dev liblapack-dev libopenmpi-dev libscalapack-mpi-dev libfftw3-dev
```

## Extract and patch VASP
```
tar -zxvf vasp.5.4.4.tar.gz
gunzip patch.5.4.4.16052018.gz
cd vasp.5.4.4/
patch -p0 < ../patch.5.4.4.16052018
```

## Create `make.include`
```
# Precompiler options
CPP_OPTIONS= -DHOST=\"LinuxGNU\" \
            -DMPI -DMPI_BLOCK=8000 -Duse_collective \
            -DscaLAPACK -DCACHE_SIZE=4000 \
            -Davoidalloc -Duse_bse_te \
            -Dtbdyn -Duse_shmem

CPP        = gcc -E -P -C -w $*$(FUFFIX) >$*$(SUFFIX) $(CPP_OPTIONS)

FC         = mpif90
FCL        = mpif90 

FREE       = -ffree-form -ffree-line-length-none 

FFLAGS     = -w
OFLAG      = -O2 -mtune=native -m64
OFLAG_IN   = $(OFLAG)
DEBUG      = -O0

LIBDIR     = /usr/lib/x86_64-linux-gnu
BLAS       = -L$(LIBDIR) -lblas
LAPACK     = -L$(LIBDIR) -llapack
BLACS      = 
SCALAPACK  = -L/usr/lib -lscalapack-openmpi $(BLACS)

LLIBS      = $(SCALAPACK) $(LAPACK) $(BLAS)

LLIBS      += -lfftw3
INCS       = -I/usr/include

OBJECTS    = fftmpiw.o fftmpi_map.o  fftw3d.o  fft3dlib.o 

OBJECTS_O1 += fftw3d.o fftmpi.o fftmpiw.o
OBJECTS_O2 += fft3dlib.o

# For what used to be vasp.5.lib
CPP_LIB    = $(CPP)
FC_LIB     = $(FC)
CC_LIB     = gcc
CFLAGS_LIB = -O
FFLAGS_LIB = -O1
FREE_LIB   = $(FREE)

OBJECTS_LIB= linpack_double.o getshmem.o 

# For the parser library
CXX_PARS   = g++  
LIBS       += parser
LLIBS      += -Lparser -lparser -lstdc++

# Normally no need to change this
SRCDIR     = ../../src
BINDIR     = ../../bin
```
## Build `std`, `gam` and `ncl` of VASP
```
make std gam ncl
```

## Adding environment variable for VASP
```
vim ~/.bashrc
```
It is depend on the location of your VASP installation. In my example:
```
PATH=/home/dell/vasp.5.4.4/build/std:$PATH
```

## Source it!
```
source ~/.bashrc
```
or if you are using zsh then:
```
source ~/.zshrc
```

## Try the magic!
```
mpirun -np 8 vasp
```



