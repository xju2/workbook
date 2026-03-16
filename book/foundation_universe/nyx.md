
## Covert PLT to HDF5
The script to submit the job `/global/homes/x/xju/m3443/usr/xju/FoundationUniverse/Nyx/Util/Converters/Plotfile2HDF5_grids/submit.sh`

To check your jobs: `sacct -j 49930712 --format=JobID,JobName,SubmitLine,WorkDir%100,State,ExitCode`

## Gimlet

```bash

```
The code is compiled at: `/global/homes/x/xju/m3443/usr/xju/FoundationUniverse/gimlet2/apps/p1d_all_axes`

To create 1D power spectrum of the Nyx simulation, we use the following command:

```bash
./p1d_all_axes.ex /pscratch/sd/z/zarija/MLHydro/L80_N4096_z3_s2.hdf5 power1d_N4096_z3_s2
```
