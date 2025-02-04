# Build ATLAS external
[ATLAS Externals](https://gitlab.cern.ch/atlas/atlasexternals)
holds the code for building all flavours of ATLAS
"externals" projects. Projects holding code not developed by ATLAS,
but used by the offline/simulation/analysis software of the
experiment. 

The following is based on my experience in installing the 
[Triton client](https://github.com/triton-inference-server/client/tree/main) 
in the ATLAS environment.

## Build at NERSC
The following instructions uses the container `docexoty/alma9-atlasos-dev`. 
The container is built with the 
[Dockerfile](https://github.com/xju2/dockers/blob/main/HEP/atlas/alma9_cpu/Dockerfile).

And I use `shifter` + the `cvmfs` module to build the ATLAS software.


### Setup the environment
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

### Environment variables for build Athena
```bash
export PATH=/cvmfs/sft.cern.ch/lcg/contrib/ninja/1.11.1/Linux-x86_64/bin:$PATH
export ATLASAuthXML=/global/cfs/cdirs/atlas/xju/data/xml
export G4PATH=/cvmfs/atlas-nightlies.cern.ch/repo/sw/main_Athena_x86_64-el9-gcc13-opt/Geant4
export NWORKERS=32
```

### Build the external packages.
`````{admonition} 1. Use the existing script to start the building.
```bash
asetup none,gcc13,cmakesetup --cmakeversion=3.30.5
export AtlasExternals_URL=https://gitlab.cern.ch/xju/atlasexternals.git
export AtlasExternals_REF=origin/new_mr_triton
rm -rf build && mkdir build
./athena/Projects/Athena/build_externals.sh -c -t Release -k "-j${NWORKERS}" 2>&1 | tee build/log.external.txt
```
Detailed build logs can be found at `build/build/AthenaExternals/BuildLogs`.

If the above failed, find the package that failed and 
continue the debugging with the following commands.
`````

````{admonition} 2. Continue the debugging.
```bash
asetup none,gcc13,cmakesetup --cmakeversion=3.30.5
cmake --build build/build/AthenaExternals --target Package_TritonClient 2>&1 | tee log.build.client
cmake --build build/build/AthenaExternals 2>&1 | tee log.build
DESTDIR=build/install cmake --install build/build/AthenaExternals 
```
Force re-configure the package.
```bash
rm -rf build/build/AthenaExternals/src/TritonClient-stamp
```
Force re-build (i.e. re-compile) the package.
```bash
rm -rf build/build/AthenaExternals/src/TritonClient-build
```
````

````{admonition} 3. Check if the environment contains all depdenencies.
```bash
vim build/install/AthenaExternals/25.0.24/InstallArea/x86_64-el9-gcc13-opt/env_setup.sh
```
````


### Build the Athena packages.

1. Use the existing script.
```bash
time ./athena/Projects/Athena/build.sh -acmi -x "-DATLAS_ENABLE_CI_TESTS=TRUE -DATLAS_EXTERNAL=${ATLASAuthXML} -DCMAKE_EXPORT_COMPILE_COMMANDS=TRUE -G Ninja" -k "-j${NWORKERS}" 2>&1 | tee build/log.build.athena.txt
```

2. If the above failed, continue the debugging with the following commands.
```bash
asetup Athena,25.0.24 --releasepath=build/install
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

3. Test the tracking as a service
Follow the instructions in [](./tracking-aas-in-athena.md) to test the tracking as a service.

## Build at lxplus
The lxplus machine is used, `aiatlasbm005`. 

```bash
setupATLAS
```
And the rest is the same as building at NERSC.
