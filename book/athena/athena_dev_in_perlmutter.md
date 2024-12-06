

After connecting to the shifter container. 


```bash
source /global/cfs/cdirs/atlas/scripts/setupATLAS.sh && setupATLAS
```
Then setup Athena release:
```bash
asetup Athena,main,here,latest
```
Then compile the code:
```bash
cmake -DCMAKE_EXPORT_COMPILE_COMMANDS=1 \
  -DATLAS_ENABLE_IDE_HELPERS=TRUE \
  -DATLAS_PACKAGE_FILTER_FILE=../athena/package_filter.txt \
  ../athena/Projects/WorkDir 

make -j 32
source x86_64-el9-gcc13-opt/setup.sh
```
