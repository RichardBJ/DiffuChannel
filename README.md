# DiffuChannel

This repository accompanies the DiffuChannel paper. DiffuChannel is a conditional diffusion model that generates single-channel patch-clamp current records. The generated data can be conditioned on an idealisation of the record simulated from a Markov chain fitted to a real seed recording, so the generated current follows the gating kinetics of the seed. A coarse envelope model and a fine model are used in two stages, and long records are generated in segments with RePaint-style context carried between them. The model is benchmarked against DeepGANnel on five phenotypes (A to E).

## Install

Python 3.11 was used. CUDA is optional, but training on CPU is slow.

```
pip install -r requirements.txt
```

The R analysis needs R and the packages listed under "Reproducing the paper".

## Quick start (app)

```
python diffusion_app.py
```

Then open http://127.0.0.1:7860. The app can train a model, fit a Markov transition matrix to a seed record, simulate an idealisation, and generate a record. The Help link opens help.html. The default data folder is `./pages`, which holds one example seed (`pages/4096L11raw.csv`).

Training writes `ion_channel_diffusion_model.pt` and dated `*_model_DDPM.pt` and `*.txt` files to the working directory. Generate writes `generated_dataset.parquet`.

For more advanced usage, the same actions can be run without the interface from a JSON config:

```
python diffusion_headless.py example_train.json
python diffusion_headless.py example_fit_matrix.json
python diffusion_headless.py example_generate.json
```

## Reproducing the paper

The trained models and generated records used in the paper are already in `batch_out/`, so the notebook can be knitted without retraining. To retrain from scratch:

```
python batch_run.py --root ./Phenotypes --out ./batch_out
python loss_history.py --dir ./batch_out
```

`batch_run.py` trains and generates for every phenotype and writes `Phenotype_X_DDPM.parquet`, `.pt` and `.txt`. `loss_history.py` draws the training-loss figures from the checkpoints.

The analysis notebook produces every figure, table and number in the paper. It reads `./Phenotypes` and `./batch_out` and writes `./benchmark_out`. It is deterministic (a single `set.seed(42)`) and takes about 75 minutes.

```
Rscript -e 'rmarkdown::render("DiffuChannel_benchmark.Rmd")'
```

R packages: rmarkdown, knitr, dplyr, tidyr, ggplot2, readr, purrr, arrow, ggh4x, scales, patchwork, tibble, ragg, Rtsne, uwot, kableExtra, and ClusterSignificance from Bioconductor:

```
BiocManager::install("ClusterSignificance")
```

## Repository layout

- `diffusion_103.py`: model, training and generation library (UNet1D, ConditionalDiffusion, two-stage generation, long-record generation).
- `diffusion_app.py`: Gradio app.
- `diffusion_headless.py`: the app actions driven by a JSON config (`example_*.json`).
- `markov_fit.py`: Baum-Welch fit of an aggregated discrete-time Markov chain to an idealisation.
- `batch_run.py`: train and generate for every phenotype.
- `loss_history.py`: training-loss figures from the checkpoints.
- `DiffuChannel_benchmark.Rmd`: benchmark analysis notebook.
- `Phenotypes/`: real seed record (`*raw*.csv`) and DeepGANnel output (`*GAN*.csv` or `.parquet`) for each phenotype.
- `batch_out/`: generated diffusion records and trained models used in the paper.
- `pages/`: example seed record for the app.
- `help.html`: help page served by the app.

## Citation

Paper in preparation.

## License

MIT. See LICENSE.
