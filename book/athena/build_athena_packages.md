# Build Athena Packages

## Setup the environment
```bash
shifter --image=docexoty/alma9-atlasos-dev:latest --module=cvmfs bash 
```

then go to the working directory and setup the ATLAS environment.
```bash
cd /pscratch/sd/x/xju/athena_dev/triton_20240923
source /global/cfs/cdirs/atlas/scripts/setupATLAS.sh 
setupATLAS
```

## Setup Athena
```bash
asetup Athena,main,latest,here
```
or if you have a privately built Athena:
```bash
asetup Athena,25.0.24 --releasepath=build/install
```

## Build packages
You need a package filterng file, `packages.txt`,
to filter out the packages you don't want to build.
One example is:
```txt
+ InnerDetector/InDetGNNTracking
+ Control/AthOnnx/AthOnnxInterfaces
+ Control/AthOnnx/AthTritonComps
+ Tracking/TrkConfig
+ InnerDetector/InDetConfig
+ InnerDetector/InDetValidation/InDetPhysValMonitoring
- .*
```
Then you can build the packages with the following command:
```bash
mkdir build_athena && cd build_athena
cmake -DATLAS_PACKAGE_FILTER_FILE=../packages.txt ../athena/Projects/WorkDir
cmake --build . 
```
