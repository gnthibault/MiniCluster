# Setup ethereum

Write the file setupethereum.sh:

```bash
#!/bin/bash
apt-get install software-properties-common
add-apt-repository -y ppa:ethereum/ethereum
add-apt-repository -y ppa:ethereum/ethereum-dev
apt-get update
apt-get install ethereum
```

And then launch on every node:

```bash
  for ((i=0; i<4; i++)); do ssh root@node$i 'bash -s' < setupethereum.sh ; done
```


wget https://github.com/weidai11/cryptopp/archive/CRYPTOPP_5_6_5.tar.gz
tar -xvf 
cd cryptopp-CRYPTOPP_5_6_5
make CXXFLAGS="-std=c++11 -fPIC"
make install PREFIX=/home/user/scratch/


apt-get install -y libjsoncpp-dev libjsonrpccpp-dev libclc-dev libleveldb-dev libcurl4-gnutls-dev libcryptopp-dev libmicrohttpd-dev

git clone https://github.com/Genoil/cpp-ethereum.git
cd ./cpp-ethereum/
mkdir build; cd build
cmake -DBUNDLE=miner ..
make -j8

or, for arm version, use
git clone --recursive https://github.com/ethereum/cpp-ethereum.git

