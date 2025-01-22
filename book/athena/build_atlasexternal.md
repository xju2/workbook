# Build ATLAS external
The following is based on my experience in stalling the 
[Triton client](https://github.com/triton-inference-server/client/tree/main) in the ATLAS environment.

## Build at NERSC
The following instructions are for building the ATLAS external packages at NERSC
using the container `docexoty/alma9-atlasos-dev`. The container is built with
the [Dockerfile](https://github.com/xju2/dockers/blob/main/HEP/atlas/alma9_cpu/Dockerfile).

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
`````{admonition} 3.1 Use the existing script to start the building.
```bash
asetup none,gcc13,cmakesetup --cmakeversion=3.30.5
export AtlasExternals_URL=https://gitlab.cern.ch/xju/atlasexternals.git
export AtlasExternals_REF=origin/new_mr_triton
rm -rf build && mkdir build
./athena/Projects/Athena/build_externals.sh -c -t Release -k "-j${NWORKERS}" 2>&1 | tee build/log.external.txt
```
If the above failed, find the package that failed and 
continue the debugging with the following commands.

````{admonition} 3.2 Continue the debugging.
```bash
asetup none,gcc13,cmakesetup --cmakeversion=3.30.5
cmake --build build/build/AthenaExternals --target Package_Gdb 2>&1 | tee log.build.Gdb
cmake --build build/build/AthenaExternals 2>&1 | tee log.build
DESTDIR=build/install cmake --install build/build/AthenaExternals 
```
````

Remove the stamp folder if you want cmake to re-configure the package.
```bash
rm -rf build/build/AthenaExternals/src/TritonClient-stamp
```
Remove the build folder if you want cmake to re-build (i.e. re-compile) the package.
```bash
rm -rf build/build/AthenaExternals/src/TritonClient-build
```

````{admonition} 3.3 Check if the environment contains all depdenencies.
```bash
vim build/install/AthenaExternals/25.0.24/InstallArea/x86_64-el9-gcc13-opt/env_setup.sh
```
````
`````


### Build the Athena packages.

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
