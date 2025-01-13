# Build atlas external

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
    export AtlasExternals_REF=origin/debug_mr_triton
    export CPLUS_INCLUDE_PATH=/cvmfs/sft.cern.ch/lcg/releases/LCG_106a/rapidjson/1.1.0/x86_64-el9-gcc13-opt/include:$CPLUS_INCLUDE_PATH
    rm -rf build && mkdir build
    ./athena/Projects/Athena/build_externals.sh -t Release -k "-j${NWORKERS}" 2>&1 | tee build/log.external.txt
    ```

    3.2 Or build the external packages manually.
    ```bash
    asetup none,gcc13,cmakesetup --cmakeversion=3.30.5
    export CPLUS_INCLUDE_PATH=/cvmfs/sft.cern.ch/lcg/releases/LCG_106a/rapidjson/1.1.0/x86_64-el9-gcc13-opt/include:$CPLUS_INCLUDE_PATH

    cmake -DCMAKE_BUILD_TYPE=RelWithDebInfo -DCTEST_USE_LAUNCHERS=TRUE  \
    -DLCG_VERSION_POSTFIX="b_ATLAS_1" \
    -S build/src/AthenaExternals/Projects/AthenaExternals \
    -B build/build/AthenaExternals

    cmake --build build/build/AthenaExternals --target Package_Gdb 2>&1 | tee log.build.Gdb

    cmake --build build/build/AthenaExternals 2>&1 | tee log.build
    DESTDIR=build/install cmake --install build/build/AthenaExternals 
    ```

4. Build the Athena packages.
```bash
time ./athena/Projects/Athena/build.sh -acmi -x "-DATLAS_ENABLE_CI_TESTS=TRUE -DATLAS_EXTERNAL=${ATLASAuthXML} -DCMAKE_EXPORT_COMPILE_COMMANDS=TRUE " -k "-j${NWORKERS}" 2>&1 | tee build/log.build.athena.txt
```
