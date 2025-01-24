# GNN-based Tracking as a Service
1. Request an interactive GPU node
```bash
srun -C "gpu&hbm80g" -q interactive -N 1 -G 1 -c 32 -t 4:00:00 -A m3443 --pty /bin/bash -l
```

2. Launch the server
```bash
cd /pscratch/sd/x/xju/ITk/ForFinalPaper/tracking-as-a-service && ./scripts/start-tritonserver.sh
```

3. Run the client
```bash
cd /pscratch/sd/x/xju/ITk/ForFinalPaper/run_athena/run && ./run_tracking.sh
```