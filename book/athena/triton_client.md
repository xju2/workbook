# All about Triton Client Saga

## Download the Triton Client

```bash
wget https://github.com/triton-inference-server/server/releases/download/v2.46.0/v2.46.0_ubuntu2204.clients.tar.gz
tar -xvzf v2.46.0_ubuntu2204.clients.tar.gz
```
Summary of tarball's md5sum:
| File | md5sum |
| --- | --- |
| v2.46.0_ubuntu2204.clients.tar.gz | dc486bfe1aaa05e6438bd08967b28f62 |

## Build binary for Triton Client

```bash
shifter --image docexoty/alma9-atlasos-dev --module cvmfs bash

cd /pscratch/sd/x/xju/athena_dev/client_build_20241202
source /global/cfs/cdirs/atlas/scripts/setupATLAS.sh
setupATLAS
asetup none,gcc13,cmakesetup --cmakeversion=3.30.5
asetup Athena,25.0.22,here
```

Then clone the client repo and build the binary with the third party libraries.

```bash
git clone -b r24.10 https://github.com/triton-inference-server/client.git
cd client 
mkdir build && cd build 
export CPLUS_INCLUDE_PATH=/cvmfs/sft.cern.ch/lcg/releases/LCG_106a/rapidjson/1.1.0/x86_64-el9-gcc13-opt/include:$CPLUS_INCLUDE_PATH
cmake .. -DTRITON_ENABLE_CC_HTTP=OFF \
        -DTRITON_ENABLE_CC_GRPC=ON \
        -DTRITON_ENABLE_PYTHON_GRPC=ON \
        -DCMAKE_INSTALL_PREFIX=../../install \
        -DTRITON_THIRD_PARTY_SRC_INSTALL_PREFIX=../../install \
        -DTRITON_COMMON_REPO_TAG=r24.10 \
        -DTRITON_CORE_REPO_TAG=r24.10 \
        -DTRITON_THIRD_PARTY_REPO_TAG=r24.10 \
        -DCMAKE_CXX_STANDARD=20 \
        -DTRITON_ENABLE_ZLIB=ON
make cc-clients -j20 
```
