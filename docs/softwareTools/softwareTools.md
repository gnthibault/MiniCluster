# Installing software tools for distributed scientific computing

## Introduction

We first want to copy our own ssh public key to the root user account so that we can login without password to every node
```bash
  for ((i=2; i<6; i++)); do ssh-copy-id root@192.168.0.$i ; done
```

Then we will create the user "user", with password "user" on every node:
```bash
  for ((i=2; i<6; i++)); do ssh root@192.168.0.$i "useradd --create-home -p $(openssl passwd -1 user) user" ; done
```

And also give them our local public key
```bash
  for ((i=2; i<6; i++)); do ssh-copy-id user@192.168.0.$i ; done
```

Then we will give all nodes the same ssh-key as the one we are currently using
```bash
  for ((i=2; i<6; i++)); do scp ~/.ssh/id_rsa user@192.168.0.$i/.ssh/ ; done
```

We are now able to run any script on all nodes with
```bash
  for ((i=2; i<6; i++)); do ssh user@192.168.0.$i 'bash -s' < local_script.sh ; done
```


## Installing slurm

## Installing scalapack

For distributed linear algebra task, we use the scalapack software stack, to do so, just run:
```bash
  sudo apt-get install libatlas-dev libopenmpi-dev libscalapack-mpi-dev -y
```

## Installing parsec

We are also interested in benchmarking the parsec library, see
[PLASMA documentation](https://bitbucket.org/icl/plasma)
and [DPLASMA documentation](https://bitbucket.org/bosilca/dplasma/wiki/Home)

to do so you will also need some additional packages, here are the steps used
to get everything to work:

# Get PLASMA
```bash
  apt-get install hwloc pkg-config
  wget http://icl.cs.utk.edu/projectsfiles/plasma/pubs/plasma_2.8.0.tar.gz
  tar -xvf ./plasma_2.8.0.tar.gz
  rm -rf ./plasma_2.8.0.tar.gz
  cd plasma_2.8.0/
```

# Get DPLASMA
```bash
  apt-get install bison flex -y
  wget http://icl.cs.utk.edu/projectsfiles/parsec/pubs/parsec_2b39da2e4087.tgz
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
