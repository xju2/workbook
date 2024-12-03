# All about Triton Client Saga

## Build binary for Triton Client

```bash
shifter --image docexoty/alma9-atlasos-dev --module cvmfs bash

cd /pscratch/sd/x/xju/athena_dev/client_build_20241202
source /global/cfs/cdirs/atlas/scripts/setupATLAS.sh
setupATLAS
asetup none,gcc13,cmakesetup --cmakeversion=3.30.5
asetup Athena,25.0.22,here
```

First install `gRPC`.
```bash
git clone -b v1.62.3 https://github.com/grpc/grpc
cd grpc
git submodule update --init
```
Build `gRPC` with `cmake` and `make`
```bash
mkdir -p cmake/build
cd cmake/build
cmake -DgRPC_INSTALL=ON -DCMAKE_BUILD_TYPE=Release \
  -DgRPC_BUILD_TESTS=OFF \
  -DgRPC_SSL_PROVIDER=package \
  -DBUILD_SHARED_LIBS=OFF \
  -DCMAKE_INSTALL_PREFIX=/pscratch/sd/x/xju/athena_dev/client_build_20241202/install \
  ../..
make -j20 install
```
Issues with `gRPC` build:
 - "per_cpu.h:70:30: internal compiler error: Segmentation fault", https://github.com/grpc/grpc/issues/33634


Then clone the client repo and build the binary

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
        -DTRITON_REPO_ORGANIZATION="https://github.com/triton-inference-server" \
        -DCMAKE_CXX_STANDARD=20 \
        -DTRITON_ENABLE_ZLIB=ON
```
