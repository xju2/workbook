## GNN4ITK as a Service


2026-01-27
* Module map as a service. `/pscratch/sd/x/xju/ITk/ForFinalPaper/taas-dev/alina_ml_pipe/models/ModuleMap/1`
* Docker container: `docker.io/alinutzal/tritonserver:2026-01-13-pymmg`

Launch the container with:
```bash
IMG_NAME="docker.io/alinutzal/tritonserver:2026-01-13-pymmg"
podman-hpc run -it --rm --gpu --ipc=host --net=host --ulimit memlock=-1 --ulimit stack=67108864 -v $PWD:$PWD -w $PWD $IMG_NAME /bin/bash
```

Verify the server by running in the container in `/models/ModuleMap/1`:
```bash
python inference.py -i selected_graph_tensor_mm.pt
```

This is the output of running IaaS.
```text
root@nid001181:/pscratch/sd/x/xju/ITk/ForFinalPaper/taas-dev/alina_ml_pipe/models/ModuleMap/1# python inference.py -i selected_graph_tensor_mm.pt
ModuleMapInferenceConfig(model_path=PosixPath('.'), module_map_pattern_path=PosixPath('.'), device='cuda', auto_cast=False, compiling=False, debug=False, device_id=0, save_debug_data=False, r_max=0.12, k_max=1000, filter_cut=0.05, filter_batches=10, cc_cut=0.01, walk_min=0.1, walk_max=0.6, embedding_node_features=['r', 'phi', 'z', 'cluster_x_1', 'cluster_y_1', 'cluster_z_1', 'cluster_x_2', 'cluster_y_2', 'cluster_z_2', 'count_1', 'charge_count_1', 'loc_eta_1', 'loc_phi_1', 'localDir0_1', 'localDir1_1', 'localDir2_1', 'lengthDir0_1', 'lengthDir1_1', 'lengthDir2_1', 'glob_eta_1', 'glob_phi_1', 'eta_angle_1', 'phi_angle_1', 'count_2', 'charge_count_2', 'loc_eta_2', 'loc_phi_2', 'localDir0_2', 'localDir1_2', 'localDir2_2', 'lengthDir0_2', 'lengthDir1_2', 'lengthDir2_2', 'glob_eta_2', 'glob_phi_2', 'eta_angle_2', 'phi_angle_2'], embedding_node_scale=[1000.0, 3.14, 1000.0, 1000.0, 1000.0, 1000.0, 1000.0, 1000.0, 1000.0, 1.0, 1.0, 3.14, 3.14, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 3.14, 3.14, 3.14, 3.14, 1.0, 1.0, 3.14, 3.14, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 3.14, 3.14, 3.14, 3.14], filter_node_features=['r', 'phi', 'z', 'cluster_x_1', 'cluster_y_1', 'cluster_z_1', 'cluster_x_2', 'cluster_y_2', 'cluster_z_2', 'count_1', 'charge_count_1', 'loc_eta_1', 'loc_phi_1', 'localDir0_1', 'localDir1_1', 'localDir2_1', 'lengthDir0_1', 'lengthDir1_1', 'lengthDir2_1', 'glob_eta_1', 'glob_phi_1', 'eta_angle_1', 'phi_angle_1', 'count_2', 'charge_count_2', 'loc_eta_2', 'loc_phi_2', 'localDir0_2', 'localDir1_2', 'localDir2_2', 'lengthDir0_2', 'lengthDir1_2', 'lengthDir2_2', 'glob_eta_2', 'glob_phi_2', 'eta_angle_2', 'phi_angle_2'], filter_node_scale=[1000.0, 3.14, 1000.0, 1000.0, 1000.0, 1000.0, 1000.0, 1000.0, 1000.0, 1.0, 1.0, 3.14, 3.14, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 3.14, 3.14, 3.14, 3.14, 1.0, 1.0, 3.14, 3.14, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 3.14, 3.14, 3.14, 3.14], gnn_node_features=['r', 'phi', 'z', 'eta', 'cluster_r_1', 'cluster_phi_1', 'cluster_z_1', 'cluster_eta_1', 'cluster_r_2', 'cluster_phi_2', 'cluster_z_2', 'cluster_eta_2'], gnn_node_scale=[1000.0, 3.14159265359, 1000.0, 1.0, 1000.0, 3.14159265359, 1000.0, 1.0, 1000.0, 3.14159265359, 1000.0, 1.0])
Path:  .
Loading checkpoint from /pscratch/sd/x/xju/ITk/ForFinalPaper/taas-dev/alina_ml_pipe/models/ModuleMap/1/MM_minmax_ignn2.ckpt
Is a recurrent GNN?: False
Use ChainedInteractionGNN2
start a warm-up run.
CUDA info: 1 GPU found
CUDA infos: free memory 38.8185 Go / 39.3812Go
Reading the Module Map: /pscratch/sd/x/xju/ITk/ForFinalPaper/taas-dev/alina_ml_pipe/models/ModuleMap/1/ModuleMap_rel24_ttbar_v9_89809evts_tol1e-10
module map triplet: nb entries = 2488597
module map doublet: nb entries = 617200
# modules in MM = 33932
Module Map loaded in memory
cpu time : 8.85859
# of modules = 33932
cpu time for memory allocation: 0.000382
------------------------
MMG: build Graphs on GPU
------------------------
  - device: 0
  - blocks: 512
  - nb_events: 1
------------------------
total tracks 3885
```

The server is running fine. Now, start the Athena client.

Start the container for Athena dev and set up env.
```bash
shifter --image=beojan/mpicuda9-2:latest --module=cvmfs,gpu

export PATH=/cvmfs/sft.cern.ch/lcg/contrib/ninja/1.11.1/Linux-x86_64/bin:$PATH
source /global/cfs/cdirs/atlas/scripts/setupATLAS.sh
setupATLAS
asetup Athena,25.0.47,here
```

Build Athena if needed.
```bash
SPARSE_BUILD_DIR=sparse_build
cmake -B ${SPARSE_BUILD_DIR} -S athena/Projects/WorkDir -DATLAS_PACKAGE_FILTER_FILE=./package_filters.txt -G "Ninja" -DCMAKE_EXPORT_COMPILE_COMMANDS=TRUE
cmake --build sparse_build -- -j 16
```

Define useful variables.
```bash
cd /pscratch/sd/x/xju/athena_dev/dev_worktree/20251003_mmAAS

TRITON_URL='nid001056'
TRITON_PORT=8001
RDO_FILENAME='/global/cfs/cdirs/m3443/data/GNN4ITK/RDOFiles/rel24_ttbar_testing/RDO.37737772._000213.pool.root.1'
OUTFILE='test.aod.pool.root'
```

Run the tracking reconstruction with MM-based GNN4ITk as a service.

```bash
TRITON_MODEL_NAME='ModuleMap'
SPFeatures="x,y,z,module_id,hit_id,r,phi,eta,cluster_r_1,cluster_phi_1,cluster_z_1,cluster_eta_1,cluster_r_2,cluster_phi_2,cluster_z_2,cluster_eta_2"
Reco_tf.py --CA 'all:True' \
  --autoConfiguration 'everything' \
  --conditionsTag 'all:OFLCOND-MC15c-SDR-14-05' \
  --geometryVersion 'all:ATLAS-P2-RUN4-03-00-00' \
  --multithreaded 'True' \
  --steering 'doRAWtoALL' \
  --digiSteeringConf 'StandardInTimeOnlyTruth' \
  --postInclude 'all:PyJobTransforms.UseFrontier' \
  --preExec "all:flags.ITk.doEndcapEtaNeighbour=True; flags.Tracking.ITkGNNPass.minClusters = [7,7,7]; flags.Tracking.ITkGNNPass.maxHoles = [4,4,2]; flags.Tracking.GNN.Triton.model = \"$TRITON_MODEL_NAME\"; flags.Tracking.GNN.Triton.url = \"$TRITON_URL\"; flags.Tracking.GNN.Triton.port = ${TRITON_PORT}; flags.Tracking.GNN.spacepointFeatures=\"${SPFeatures}\" " \
  --preInclude 'all:Campaigns.PhaseIIPileUp200' 'InDetConfig.ConfigurationHelpers.OnlyTrackingPreInclude' 'InDetGNNTracking.InDetGNNTrackingFlags.gnnTritonValidation' \
  --inputRDOFile="${RDO_FILENAME}" \
  --outputAODFile="OUTFILE" --athenaopts='--loglevel=INFO' --maxEvents 2 --perfmon fullmonmt
```

Run Double Metric Learning inference:
```bash
TRITON_MODEL_NAME='DoubleMetricLearning'
Reco_tf.py --CA 'all:True' \
  --autoConfiguration 'everything' \
  --conditionsTag 'all:OFLCOND-MC15c-SDR-14-05' \
  --geometryVersion 'all:ATLAS-P2-RUN4-03-00-00' \
  --multithreaded 'True' \
  --steering 'doRAWtoALL' \
  --digiSteeringConf 'StandardInTimeOnlyTruth' \
  --postInclude 'all:PyJobTransforms.UseFrontier' \
  --preExec "all:flags.ITk.doEndcapEtaNeighbour=True; flags.Tracking.ITkGNNPass.minClusters = [7,7,7]; flags.Tracking.ITkGNNPass.maxHoles = [4,4,2]; flags.Tracking.GNN.Triton.model = \"$TRITON_MODEL_NAME\"; flags.Tracking.GNN.Triton.url = \"$TRITON_URL\"; flags.Tracking.GNN.Triton.port = ${TRITON_PORT} " \
  --preInclude 'all:Campaigns.PhaseIIPileUp200' 'InDetConfig.ConfigurationHelpers.OnlyTrackingPreInclude' 'InDetGNNTracking.InDetGNNTrackingFlags.gnnTritonValidation' \
  --inputRDOFile="${RDO_FILENAME}" \
  --outputAODFile="OUTFILE" --athenaopts='--loglevel=INFO' --maxEvents 2 --perfmon fullmonmt
```

MM with SPIN system

Or using the open URL
```bash
TRITON_URL='iaasdemo.ml4phys.com'
TRITON_PORT=443

TRITON_URL='amsc-d3.ml4phys.com'
```


```bash
TRITON_MODEL_NAME='ModuleMap'
SPFeatures="x,y,z,module_id,hit_id,r,phi,eta,cluster_r_1,cluster_phi_1,cluster_z_1,cluster_eta_1,cluster_r_2,cluster_phi_2,cluster_z_2,cluster_eta_2"
Reco_tf.py --CA 'all:True' \
  --autoConfiguration 'everything' \
  --conditionsTag 'all:OFLCOND-MC15c-SDR-14-05' \
  --geometryVersion 'all:ATLAS-P2-RUN4-03-00-00' \
  --multithreaded 'True' \
  --steering 'doRAWtoALL' \
  --digiSteeringConf 'StandardInTimeOnlyTruth' \
  --postInclude 'all:PyJobTransforms.UseFrontier' \
  --preExec "all:flags.ITk.doEndcapEtaNeighbour=True; flags.Tracking.ITkGNNPass.minClusters = [7,7,7]; flags.Tracking.ITkGNNPass.maxHoles = [4,4,2]; flags.Tracking.GNN.Triton.model = \"$TRITON_MODEL_NAME\"; flags.Tracking.GNN.Triton.url = \"$TRITON_URL\"; flags.Tracking.GNN.Triton.port = ${TRITON_PORT}; flags.Tracking.GNN.spacepointFeatures=\"${SPFeatures}\" " \
  --preInclude 'all:Campaigns.PhaseIIPileUp200' 'InDetConfig.ConfigurationHelpers.OnlyTrackingPreInclude' 'InDetGNNTracking.InDetGNNTrackingFlags.gnnTritonValidation' \
  --inputRDOFile="${RDO_FILENAME}" \
  --outputAODFile="OUTFILE" --athenaopts='--loglevel=INFO' --maxEvents 2 --perfmon fullmonmt
```
