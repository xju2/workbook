---
tags: workbook, fundra
---

# Commands of running the Foundation Universe

The repository is located: https://github.com/xju2/foundational_universe.

## 2025/08/14
Organize the tokens in a 3D structure.
Need to reprocess the data using a chunk size of 256. During the training, use batch size of 1.
```{code-block} bash
:class: wrap

fundra_preprocess -f src/fundra/configs/resolved/vqvae/v0.6.0.yaml -c datamodule.num_workers=8
```

Evaluation of the training of `v0.6.0`.
```{code-block} bash
:class: wrap

fundra_eval_tokens -f src/fundra/configs/resolved/vqvae/v0.6.0.yaml -c /pscratch/sd/x/xju/code/foundational_universe/logs/vqvae/v0.6.0/checkpoints/best-qyd9j9us-63-259400.ckpt -o logs/fundra_tokenization_test/v0.6.0/eval-v7 -n 10 -j 2
```

## 2025/08/11
```{code-block} bash
:class: wrap

fundra_eval_tokens -f src/fundra/configs/resolved/vqvae/v0.5.0.yaml -c /pscratch/sd/x/xju/code/foundational_universe/logs/vqvae/v0.5.0/checkpoints/best-wlsbpiyl-0-1800.ckpt -o logs/fundra_tokenization_test/v0.5.0/eval-v4 -n 10 -j 2

fundra_eval_tokens -f src/fundra/configs/resolved/vqvae/v0.4.1.yaml -c /pscratch/sd/x/xju/code/foundational_universe/logs/vqvae/v0.4.1/checkpoints/best-ldd04iq7-2-20800.ckpt -o logs/fundra_tokenization_test/v0.4.1/eval-v2 -n 800 -j 40
```

## 2025/08/09
Evaluate the training of `v0.4.5`, which is when I am using the new scaling method.
```{code-block} bash
:class: wrap

fundra_eval_tokens -f src/fundra/configs/resolved/vqvae/v0.4.5.yaml -c /pscratch/sd/x/xju/code/foundational_universe/logs/vqvae/v0.4.5/checkpoints/best-smv6apty-0-1400.ckpt -o logs/fundra_tokenization_test/v0.4.5/eval-v1 -n 1
```
The performance is even worse than `v0.4.4`. All embeddings match to the same token.
Maybe the codebook size (4096) is too large. Test codebook size of 1024 in `v0.4.6`.

Prepare the small dataset for testing different ideas.
```{code-block} bash
:class: wrap

fundra_analyze_dataset /pscratch/sd/z/zarija/MLHydro/L80_N512_z3_s1.hdf5 --output /global/cfs/cdirs/m3443/usr/xju/Fundra/data/scaled_features_physics/.cache/L80_N512_z3_s1_cache/cosmo_fields.csv -w 6

fundra_analyze_dataset /pscratch/sd/z/zarija/MLHydro/L80_N512_z3_s2.hdf5 --output /global/cfs/cdirs/m3443/usr/xju/Fundra/data/scaled_features_physics/.cache/L80_N512_z3_s2_cache/cosmo_fields.csv -w 6
```

`v0.4.7` disables the `rotate_trick` and enables `straight_through`.
However, the loss goes down at the same rate as `v0.4.6`, almost exactly the same...
Not sure why.

Fixed the warning of gradient stride does not match the tensor stride.

## 2025/08/08
The most trained version `v0.4.4` in the W&B project `fundra_tokenization_v2`. I trained it for 100 hours with 16 GPUs.

```{code-block} bash
:class: wrap

fundra_eval_tokens -f src/fundra/configs/resolved/vqvae/v0.4.4.yaml -c /pscratch/sd/x/xju/code/foundational_universe/logs/vqvae/v0.4.4/checkpoints/best-gzjnln47-4-38800.ckpt -o logs/fundra_tokenization_test/v0.4.4/eval-v1 -n 50
```

The scaling part turns out to be not so good. See https://github.com/xju2/foundational_universe/issues/16. Rework the scaling.

```{code-block} bash
:class: wrap

get_node

fundra_analyze_dataset /pscratch/sd/z/zarija/MLHydro/L80_N4096_z3_s1.hdf5 --output /global/cfs/cdirs/m3443/usr/xju/Fundra/data/scaled_features_using_physics_scaling/.cache/L80_N4096_z3_s1_cache/dataset_statistics.csv -w 128

mkdir -p /global/cfs/cdirs/m3443/usr/xju/Fundra/data/scaled_features_using_physics_scaling/.cache/L80_N4096_z3_s2_cache

fundra_analyze_dataset /pscratch/sd/z/zarija/MLHydro/L80_N4096_z3_s2.hdf5 --output /global/cfs/cdirs/m3443/usr/xju/Fundra/data/scaled_features_using_physics_scaling/.cache/L80_N4096_z3_s2_cache/dataset_statistics.csv -w 128
```

check out the `fix-16`.

## 2025/08/1
Evaluate the training of `v0.4.1`
```{code-block} bash
:class: wrap

fundra_eval_tokens -f src/fundra/configs/resolved/vqvae/v0.4.1.yaml -c /pscratch/sd/x/xju/code/foundational_universe/logs/vqvae/v0.4.1/checkpoints/best-ldd04iq7-2-19200.ckpt -o logs/fundra_tokenization_test/v0.4.1/eval-v1
```
and re-evaluate the `v0.3.1` with the new x-axis range definition.

```{code-block} bash
:class: wrap

fundra_eval_tokens -f logs/fundra_tokenization_test/v0.3.1/.hydra/config.yaml -c logs/fundra_tokenization_test/v0.3.1/checkpoints/best-gl91p2s2-1-26650.ckpt -o logs/fundra_tokenization_test/v0.3.1/eval-v2
```

## 2025/07/31

Evaluate the training of `v0.4.0`.
```{code-block} bash
:class: wrap

fundra_eval_tokens -f src/fundra/configs/resolved/vqvae/v0.4.0.yaml -c /pscratch/sd/x/xju/FoundationUniverse/foundational_universe_dev/logs/vqvae/v0.4.0/checkpoints/best-i03giyf6-4-35400.ckpt -o logs/fundra_tokenization_test/v0.4.0/eval-v1
```
It turns out the `vector-quantize-torch` returns only the commit loss, no embedding loss. That means the embeddings were never updated. That explains why the token utilization was never improved.

Retrain the same model with the embedding loss.

```{code-block} bash
:class: wrap

fundra_create_train_config 'experiment=tokenization_N4096_exlib_v0.4.0' \
  'datamodule.batch_size=1024' \
  'datamodule.num_workers=1' \
  'stage=resume' \
  'version_name=v0.4.1' \
  'trainer=ddp' \
  'trainer.devices=4' \
  'trainer.num_nodes=4' \
  '+trainer.max_steps=10_000_000' \
  'trainer.max_epochs=-1' \
  'trainer.check_val_every_n_epoch=null' \
  'trainer.val_check_interval=50' \
  'trainer.detect_anomaly=true' \
  'trainer.enable_progress_bar=false' \
  'model.scheduler.patience=3' \
  'model.scheduler.factor=0.95' \
  'model.scheduler_frequency=100' \
  'callbacks.model_checkpoint.every_n_train_steps=100'
```
which will create the file `src/fundra/configs/resolved/vqvae/v0.4.1.yaml`. And then I train the model:

```bash
./scripts/interactive_continous.sh scripts/config_train_v0.4.1.json
```


## 2025/07/28

```bash
fundra_create_train_config 'experiment=tokenization_N4096_exlib_v0.4.0' \
  'datamodule.batch_size=1024' \
  'datamodule.num_workers=2' \
  'stage=fit' \
  'version_name=v0.4.0' \
  'trainer=ddp' \
  'trainer.devices=4'
```
Then run the distributed training with:
```bash
./scripts/interactive_training.sh
```
But it did not work.

```{code-block} bash
:class: wrap

fundra_train_multigpu -f src/fundra/configs/resolved/vqvae/v0.4.0.yaml -c stage=fit trainer.devices=1 compile=False trainer.fast_dev_run=True datamodule.batch_size=1024

fundra_train_multigpu -f src/fundra/configs/resolved/vqvae/v0.4.0.yaml -c stage=resume datamodule.batch_size=1024 datamodule.num_workers=2
```

Some unit tests.
```bash
python -m pytest tests/test_index_manager.py -v
python -m pytest -s tests/test_universe_patch_datamodule.py -v
python --cov=src --cov-report=xml
```

Run the training with 16 GPUs, i.e. 4 nodes with 4 GPUs each. Need GPU with 80 GB.
```bash
./scripts/interactive_continous.sh scripts/config_train_v0.4.0.json
```
* [x] Next time set the `val_check_interval` to 600 in this 16 GPU training;
otherwise, it will take too long to see the validation results.
* [x] update the `utils.log_hyperparameters` to log some selective hyperparmeters
for displaying in the wandb dashboard.

