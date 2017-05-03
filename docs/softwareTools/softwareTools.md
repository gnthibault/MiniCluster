# Installing software tools for distributed scientific computing

## Introduction

We first want to copy our own ssh public key to the root user account so that we can login without password to every node
```bash
  for ((i=4; i<8; i++)); do ssh-copy-id root@192.168.0.$i ; done
```

Then we will create the user "user", with password "user" on every node:
```bash
  for ((i=4; i<8; i++)); do ssh root@192.168.0.$i "useradd --create-home -p \$(openssl passwd -1 user) -s /bin/bash user" ; done
```

And also give them our local public key, which can also be interesting to checkout private github repo
```bash
  for ((i=4; i<8; i++)); do ssh-copy-id user@192.168.0.$i ; done
```

Then we will generate a passphrase-less rsa keypair on the host node, and copy it to every node:
```bash
  ssh-keygen -t rsa
  for ((i=4; i<8; i++)); do scp ./id_rsa* user@192.168.0.$i:/home/user/.ssh/ ; done
```

Now, to enable passwordless ssh, we should do, from the first node for example

```bash
    for ((i=5; i<8; i++)); do ssh-copy-id user@192.168.0.$i ; done
```
Excellet, now every node can ssh to any other one without password

We are now also able to run any script on all nodes with
```bash
  for ((i=4; i<8; i++)); do ssh user@192.168.0.$i 'bash -s' < local_script.sh ; done
```

## Host file
As is it probably easier to manipulate hostname than ip address, we will use a host file in order to ease our system administration job. To do so we will use the following script (named hosts.sh):

```bash
  #!/bin/bash

  echo "
  #MPI CLUSTER SETUP
  192.168.0.4    node0
  192.168.0.5    node1
  192.168.0.6    node2
  192.168.0.7    node3" >> /etc/hosts
```

And then, launch it to each node:

```bash
  for ((i=4; i<8; i++)); do ssh root@node$i 'bash -s' < hosts.sh ; done
```

## Setuping NFS
### Common part
Both server and clients will need the common packages, and create a dedicated directory scratch that will be shared across nodes
```bash
  for ((i=0; i<4; i++)); do ssh user@node$i "mkdir /home/user/scratch" ; done
```
And install the right packages
```bash
  for ((i=0; i<4; i++)); do ssh root@node$i "apt-get install nfs-common -y"; done
```

### Server side

In order to share datafiles, we are interested in setting up a network file system, but we have to choose one node (192.168.0.4) that will be the server of our nfs:
```bash
  ssh root@node0 "apt-get install nfs-kernel-server; echo \"/home/user/scratch *(rw,sync,no_root_squash,no_subtree_check)\" >> /etc/exports; exportfs -a; service nfs-kernel-server restart"
```

### Client side
Every other nodes will have to mount the nfs:
```bash
  for ((i=1; i<4; i++)); do ssh root@node$i "echo \"node0:/home/user/scratch /home/user/scratch nfs\" >> /etc/fstab; mount -a"; done
```

you can also mount the nfs on your own computer with
```bash
  sudo mount -t nfs node0:/home/user/scratch ~/scratch; sudo chown $USER:$USER ~/scratch/
```

## Installing slurm
Nothing here for now

## Installing Eigen

take a look at this page: http://eigen.tuxfamily.org/index.php?title=Main_Page
login to node0 for instance, and do
```bash
  wget http://bitbucket.org/eigen/eigen/get/3.3.3.tar.bz2
  tar -xvf ./3.3.3.tar.bz2
  cd ; mkdir build; cd build; cmake ..
  make 
```

Then install to all nodes:
```bash
  for ((i=0; i<4; i++)); do ssh root@node$i "apt-get install cmake libboost-all-dev -y" ; done
  for ((i=0; i<4; i++)); do ssh root@node$i "cd /home/user/scratch/projects/eigen-eigen-67e894c6cd8f/build ; make install" ; done
```
## Installing scalapack

For distributed linear algebra task, we use the scalapack software stack, to do so, just run:
```bash
  for ((i=0; i<4; i++)); do ssh root@node$i "apt-get install libatlas-base-dev libatlas-dev libopenmpi-dev libscalapack-mpi-dev -y"; done
```


## installing hpx

It is first recommended to get boost, hwloc and jemalloc
```bash
  apt-get install libboost-dev libhwloc-dev libjemalloc-dev  -y
```

Then clone hpx git repository

```bash
  git clone https://github.com/STEllAR-GROUP/hpx.git
  cd hpx; make build; cd build
```

Then build


```bash
cmake \
 -DCMAKE_BUILD_TYPE=Release \
 -DCMAKE_INSTALL_PREFIX=/usr/local \
 -DCMAKE_CXX_FLAGS=-std=c++14 \
 -DCMAKE_EXE_LINKER_FLAGS=-dynamic \
 -DHPX_WITH_HWLOC=ON \
 -DHPX_WITH_MALLOC=JEMALLOC \
 -DHPX_WITH_TESTS=OFF \
 -DHPX_WITH_EXAMPLES=OFF \
 -DHPX_WITH_PARCELPORT_MPI=ON -DHPX_WITH_PARCELPORT_MPI_MULTITHREADED=ON \
 -DHPX_WITH_THREAD_IDLE_RATES=ON \
 .. 
```

Then build on node0, then install on all nodes.


## Installing parsec

We are also interested in benchmarking the parsec library, see
[PLASMA documentation](https://bitbucket.org/icl/plasma)
and [DPLASMA documentation](https://bitbucket.org/bosilca/dplasma/wiki/Home)

to do so you will also need some additional packages, here are the steps used
to get everything to work:

# Get PLASMA
```bash
  apt-get install -y hwloc pkg-config
  wget http://icl.cs.utk.edu/projectsfiles/plasma/pubs/plasma_2.8.0.tar.gz
  tar -xvf ./plasma_2.8.0.tar.gz
  rm -rf ./plasma_2.8.0.tar.gz
  cd plasma_2.8.0/
```

And modify the make.inc as follow:
```
echo
"# Those variables have to be changed accordingly!
# Compilers, linker/loaders, the archiver, and their options.

# Install directory
prefix    = /usr/local/

# To speed up the compilation
MAKE      = make -j8

CC        = gcc
FC        = gfortran
LOADER    = gcc -dynamic

ARCH      = ar
ARCHFLAGS = cr
RANLIB    = ranlib

CFLAGS    = -O2 -DADD_ 
FFLAGS    = -O2 
LDFLAGS   = -O2

# To compile Fortran 90 interface
PLASMA_F90 =  1

# To compile Plasma with EZTrace library in order to trace events
# (Don’t forget to set correctly PKG_CONFIG_PATH to make `pkg-config --libs eztrace` works)
PLASMA_TRACE= 0

# By sequential kernel
# CFLAGS += -DTRACE_BY_KERNEL

# By parallel plasma function (Works only with dynamic scheduler)
# CFLAGS += -DTRACE_BY_FUNCTION

# Blas Library
LIBBLAS     = -L/usr/lib -lblas -lgfortran
# CBlas library
LIBCBLAS    = -L/usr/lib -lcblas
# lapack and tmg library (lapack is included in acml)
LIBLAPACK   = -L/usr/lib -llapack -ltmglib 
INCCLAPACK  = -I/usr/include/
LIBCLAPACK  = -L/usr/lib -llapacke" > ./make.inc
```

Then
```
make; make install
```

# Get DPLASMA
```bash
  apt-get install bison flex -y
  git clone https://bitbucket.org/icldistcomp/parsec.git  
  cd parsec; mkdir build; cd build; cmake .. -DCOREBLAS_DIR=/usr/local; make -j8; make install
```

# Network

This is unrelated, but in case you need to give you box (connected through ethernet cable) access to network, using you laptop wifi,
just use ip masquerading thanks to netfilter module of the kernel:

```bash
  sudo iptables -F
  sudo iptables -X
  sudo iptables -A FORWARD -o enp0s31f6 -j ACCEPT
  sudo iptables -A FORWARD -i enp0s31f6 -j ACCEPT
  sudo iptables -t nat -A POSTROUTING -o wlp4s0 -j MASQUERADE
  #check with
  sudo iptables -L -n -v
```
