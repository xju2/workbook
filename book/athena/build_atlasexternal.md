# Build ATLAS external

## Build at NERSC
The following instructions are for building the ATLAS external packages at NERSC
using the container `docexoty/alma9-atlasos-dev`. The container is built with
the [Dockerfile](https://github.com/xju2/dockers/blob/main/HEP/atlas/alma9_cpu/Dockerfile).

1. Setup the environment
```bash
shifter --image=docexoty/alma9-atlasos-dev:latest --module=cvmfs bash 
```

then go to the working directory and setup the ATLAS environment.
```bash
cd /pscratch/sd/x/xju/athena_dev/triton_20240923
source /global/cfs/cdirs/atlas/scripts/setupATLAS.sh 
setupATLAS
```

See document about [](./checkout-athena.md) checking out Athena.

2. Environment variables for build Athena
```bash
export PATH=/cvmfs/sft.cern.ch/lcg/contrib/ninja/1.11.1/Linux-x86_64/bin:$PATH
export ATLASAuthXML=/global/cfs/cdirs/atlas/xju/data/xml
export G4PATH=/cvmfs/atlas-nightlies.cern.ch/repo/sw/main_Athena_x86_64-el9-gcc13-opt/Geant4
export NWORKERS=32
```

3. Build the external packages.

    3.1 Use the existing script.
    ```bash
    asetup none,gcc13,cmakesetup --cmakeversion=3.30.5
    export AtlasExternals_URL=https://gitlab.cern.ch/xju/atlasexternals.git
    export AtlasExternals_REF=origin/new_mr_triton
    rm -rf build && mkdir build
    ./athena/Projects/Athena/build_externals.sh -c -t Release -k "-j${NWORKERS}" 2>&1 | tee build/log.external.txt
    ```

    3.2 Or build the external packages manually.
    ```bash
    asetup none,gcc13,cmakesetup --cmakeversion=3.30.5

    cmake -DCMAKE_BUILD_TYPE=RelWithDebInfo -DCTEST_USE_LAUNCHERS=TRUE \
        -S build/src/AthenaExternals/Projects/AthenaExternals \
        -B build/build/AthenaExternals \
        -DLCG_VERSION_NUMBER=106 \
        -DLCG_VERSION_POSTFIX="b_ATLAS_1" \
        -DATLAS_GAUDI_SOURCE="URL;https://gitlab.cern.ch/atlas/Gaudi/-/archive/v39r1.001/Gaudi-v39r1.001.tar.gz;URL_MD5;ac2bdcde14c2feb7684e34d6e7879db8" \
        -DATLAS_ACTS_SOURCE="URL;https://github.com/acts-project/acts/archive/refs/tags/v38.2.0.tar.gz;URL_HASH;SHA256=90f23bd409a153fee0a78d07d230996bfe1c8ccdc8753798a594456a8e41d28e" \
        -DATLAS_GEOMODEL_SOURCE="URL;https://gitlab.cern.ch/GeoModelDev/GeoModel/-/archive/6.7.0/GeoModel-6.7.0.tar.bz2;URL_MD5;450616aa33f97857aad3c7cbe1ff74fd" \
        -DATLAS_VECMEM_SOURCE="URL;http://cern.ch/atlas-software-dist-eos/externals/vecmem/v1.5.0.tar.gz;https://github.com/acts-project/vecmem/archive/refs/tags/v1.5.0.tar.gz;URL_MD5;3cc5a3bb14b93f611513535173a6be28" \
        -DATLAS_GEANT4_USE_LTO=TRUE \
        -DATLAS_VECGEOM_USE_LTO=TRUE

    cmake --build build/build/AthenaExternals --target Package_Gdb 2>&1 | tee log.build.Gdb

    cmake --build build/build/AthenaExternals 2>&1 | tee log.build
    DESTDIR=build/install cmake --install build/build/AthenaExternals 
    ```

4. Build the Athena packages.

    4.1 Use the existing script.
    ```bash
    time ./athena/Projects/Athena/build.sh -cmi -x "-DATLAS_ENABLE_CI_TESTS=TRUE -DATLAS_EXTERNAL=${ATLASAuthXML} -DCMAKE_EXPORT_COMPILE_COMMANDS=TRUE " -k "-j${NWORKERS}" 2>&1 | tee build/log.build.athena.txt
    ```

    4.2 If the above failed, continue the debugging with the following commands.
    ```bash
    asetup Athena,25.0.24 --releasepath=build/install --siteroot=/cvmfs/atlas-nightlies.cern.ch/repo/sw/main_Athena_x86_64-el9-gcc13-opt
    ```
    Maybe add the following environment variables.
    ```bash
    export DATAPATH=$DATAPATH:/cvmfs/atlas-nightlies.cern.ch/repo/sw/main_Athena_x86_64-el9-gcc13-opt/atlas/offline/ReleaseData/v20
    ```
    Then you can use the [](./build_athena_packages.md) instructions to debug the failed packages.

Possible issues
- adding `export CPLUS_INCLUDE_PATH=xxxx:$CPLUS_INCLUDE_PATH` causes the Ddb fails to build.
- checker_gccplugins not found.
- "Could not find nvcc, please set CUDAToolkit_ROOT."

## Build at lxplus
The lxplus machine is used, `aiatlasbm005`. 

```bash
setupATLAS
```
And the rest is the same as building at NERSC.
