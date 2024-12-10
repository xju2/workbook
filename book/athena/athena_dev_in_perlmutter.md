
# Setup Development Environment in Perlmutter

> Last update: {sub-ref}`today` | {sub-ref}`wordcount-words` words | {sub-ref}`wordcount-minutes` min read

<!-- ```{contents} Table of Contents
:depth: 3
``` -->

See document about [](./Development-in-Shifter-using-VSCode.md).
See my citation {cite}`ExaTrkX:2021abe`.

## Build Athena

```{code-block} bash
source /global/cfs/cdirs/atlas/scripts/setupATLAS.sh && setupATLAS
```
Then setup Athena release:
```bash
asetup Athena,main,here,latest
```
Then compile the code:
```{code-block} bash
:caption: Configure and build Athena.
cmake -DCMAKE_EXPORT_COMPILE_COMMANDS=1 \
  -DATLAS_ENABLE_IDE_HELPERS=TRUE \
  -DATLAS_PACKAGE_FILTER_FILE=../athena/package_filter.txt \
  ../athena/Projects/WorkDir 

make -j 32
source x86_64-el9-gcc13-opt/setup.sh
```


```{bibliography}
```