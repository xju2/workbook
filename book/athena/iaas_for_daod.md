## Evaluate the dynamic batching

Start the server...
```bash
registry_cli start-triton model_manifests/daod_one_model.yaml --accelerator KIND_CPU --model-outdir model_one
```

```bash
TRITON_RELEASE=24.05
podman-hpc run -it --gpu --net=host -v $PWD:$PWD -w $PWD nvcr.io/nvidia/tritonserver:${TRITON_RELEASE}-py3-sdk

perf_analyzer -m "BTagging_network_8085e6c5717c" \
  --service-kind triton \
  -i gRPC \
  -u iaasdemo.ml4phys.com:443 \
  --ssl-grpc-use-ssl \
  --input-data "random" \
  --shape track_features:20,24 \
  --shape jet_features:1,2 \
  --shape flow_features:10,5 \
  --shape electron_features:10,28 \
  --sync \
  --concurrency-range 1:16 \
  --measurement-interval 1000 \
  --measurement-request-count 10000
```
