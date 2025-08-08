---
tags: workbook, fundra
---

# Commands of running the Foundation Universe

The repository is located: https://github.com/xju2/foundational_universe.
## 2025/08/08
The most trained version `v0.4.4` in the W&B project `fundra_tokenization_v2`. I trained it for 100 hours with 16 GPUs.

```bash!
fundra_eval_tokens -f src/fundra/configs/resolved/vqvae/v0.4.4.yaml -c /pscratch/sd/x/xju/code/foundational_universe/logs/vqvae/v0.4.4/checkpoints/best-gzjnln47-4-38800.ckpt -o logs/fundra_tokenization_test/v0.4.4/eval-v1 -n 50
```

## 2025/08/1
Evaluate the training of `v0.4.1`
```bash!
fundra_eval_tokens -f src/fundra/configs/resolved/vqvae/v0.4.1.yaml -c /pscratch/sd/x/xju/code/foundational_universe/logs/vqvae/v0.4.1/checkpoints/best-ldd04iq7-2-19200.ckpt -o logs/fundra_tokenization_test/v0.4.1/eval-v1
```
and re-evaluate the `v0.3.1` with the new x-axis range definition.

```bash!
fundra_eval_tokens -f logs/fundra_tokenization_test/v0.3.1/.hydra/config.yaml -c logs/fundra_tokenization_test/v0.3.1/checkpoints/best-gl91p2s2-1-26650.ckpt -o logs/fundra_tokenization_test/v0.3.1/eval-v2
```

## 2025/07/31

Evaluate the training of `v0.4.0`.
```bash!
fundra_eval_tokens -f src/fundra/configs/resolved/vqvae/v0.4.0.yaml -c /pscratch/sd/x/xju/FoundationUniverse/foundational_universe_dev/logs/vqvae/v0.4.0/checkpoints/best-i03giyf6-4-35400.ckpt -o logs/fundra_tokenization_test/v0.4.0/eval-v1
```
It turns out the `vector-quantize-torch` returns only the commit loss, no embedding loss. That means the embeddings were never updated. That explains why the token utilization was never improved.

Retrain the same model with the embedding loss.

```bash!
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

```bash!
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

