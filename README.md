# LPI-RTDETR

LPI-RTDETR injects two low-dimensional luminance cues into the P3--P5 features
of RT-DETRv2-R18. The RGB detection path is unchanged. A low-resolution
Multinex-Nano branch supplies luminance variation, while an analytical darkness
map marks underexposed regions.

## Main result

All values below use the fixed ExDark split, 640x640 single-scale inference,
and no test-time augmentation.

| model | COCO AP | AP50 | small-object AP | FP16 FPS |
|---|---:|---:|---:|---:|
| RT-DETRv2-R18 | 51.843 | 77.941 | 15.557 | 66.92 |
| LPI-RTDETR (E2) | 51.780 | 77.857 | **16.875** | 52.41 |

The full five-row ablation is stored in
`results/official/same_protocol_comparison.csv`.

## Repository layout

- `models.py`: baseline, pixel cascade, LPI, gated, and regularized variants.
- `reproduce.py`: resumable training, evaluation, benchmarking, and analysis.
- `pipeline.py`: data validation, training stages, COCO evaluation, and timing.
- `analysis.py`: brightness strata, paired bootstrap, and confusion matrices.
- `data/annotations`: the exact 3000/1800/2563 COCO-format split.
- `results`: immutable metrics and compact logs from the reported run.
- `third_party`: RT-DETRv2 and Multinex source required by the experiment.

The ExDark images and large checkpoints are not stored in Git.

## Environment

```bash
conda env create -f environment.yml
conda activate lpi-rtdetr
python tools/download_weights.py
```

The original run used Python 3.10, PyTorch 2.0.1, CUDA 11.8, and an RTX 3080
20 GB. Newer PyTorch versions may work but were not used for the reported run.

## ExDark data

Download ExDark from its official source and point to the directory that
contains the category folders. Do not rename files. Verify the fixed split:

```bash
python tools/prepare_data.py --images /path/to/ExDark/images
```

Expected counts are 3000 training, 1800 validation, and 2563 test images over
12 categories. The command also checks missing files and split overlap.

## Quick check

The repository check does not require a GPU:

```bash
python tools/check_repo.py
```

The model/data smoke test performs one real forward and backward pass for all
five variants and requires CUDA:

```bash
python reproduce.py --images /path/to/ExDark/images --dry-run
```

## Full reproduction

```bash
python reproduce.py --images /path/to/ExDark/images
```

The run performs 30 common epochs, followed by 80 epochs for each variant.
Completed stages are reused when the same command is started again. Outputs go
to `runs/exdark` unless `--output` is supplied.

Important fixed choices:

- batch size 16; validation batch size 8;
- seeds 2026 and 2027;
- validation COCO AP selects checkpoints;
- test is evaluated after checkpoint selection;
- primary metric is COCO AP@[0.50:0.95];
- inference is single-scale without TTA.

## Weights

`tools/download_weights.py` downloads the official RT-DETRv2-R18 COCO
checkpoint and verifies its SHA256 hash. The small Multinex-Nano checkpoint is
included with its original noncommercial license. Fine-tuned ExDark weights
should be attached to a GitHub Release rather than committed to the repository;
the expected filenames and checksums belong in `release_assets/SHA256SUMS.txt`.

After downloading the E2 release checkpoint, reproduce its test metrics with:

```bash
python tools/evaluate.py --images /path/to/ExDark/images \
  --variant lpi --checkpoint release_assets/lpi_rtdetr_e2_best.pth
```

## Naming

The public names are `lpi` (E2), `gated` (E3), and `regularized` (E4). Old local
experiments used `prior` for E2 and `v2` for E4. The archived CSV files retain
those historical keys where changing them would alter the original record.

## License

This repository is for research and teaching. Third-party terms are listed in
`THIRD_PARTY_NOTICES.md`. Multinex is noncommercial software.

