# ENG 2440 Assignment 2: Tumor Segmentation in MRI Volumes

Patch-based 3D U-Net that segments the whole tumor from four-channel brain MRI
(MSD Task01_BrainTumour). Uniform and tumor-aware patch sampling are compared under
the same training budget, then full volumes are rebuilt with sliding-window inference.

## Folder layout

```
MedAI A2/
|-- Task01_BrainTumour/        # dataset (not in git)
|   |-- imagesTr/              # 484 four-channel MRI volumes (.nii.gz)
|   |-- labelsTr/              # 484 label volumes (.nii.gz)
|   |-- imagesTs/              # unlabelled official test set, NOT used
|   |-- dataset.json
|-- assignment2_split.csv      # case-level train / validation / test split (given)
|-- Assignment2_Student.ipynb  # all code, run top to bottom (saved with outputs)
|-- outputs/
|   |-- figures/               # every figure in the report (.png)
|   |-- results/               # every table in the report (.csv)
|   |-- checkpoints/           # model weights (not in git)
|-- report/                    # LaTeX report: main.tex + figures/ (upload the folder to Overleaf)
|-- cache/                     # preprocessed volumes (not in git)
|-- docs/                      # assignment brief (not in git)
|-- downloads/                 # original .tar archive (not in git)
|-- requirements.txt
|-- ai_use_declaration.txt
```

## Setup

Windows, Python 3.11.

```powershell
python -m venv .venv
.venv\Scripts\python.exe -m pip install torch --index-url https://download.pytorch.org/whl/cu128
.venv\Scripts\python.exe -m pip install -r requirements.txt
.venv\Scripts\python.exe -m ipykernel install --user --name medai-a2 --display-name "MedAI A2 (.venv)"
```

The CUDA 12.8 build of PyTorch is required for RTX 50-series GPUs.
Exact versions used are pinned in `requirements-lock.txt`.

## Data

1. Download `Task01_BrainTumour.tar` from
   https://msd-for-monai.s3-us-west-2.amazonaws.com/Task01_BrainTumour.tar (about 7.6 GB).
2. Extract it next to the notebook.
3. If you extract it somewhere else, change `DATA_ROOT` in Section 0.1 of the notebook. Nothing else needs to change.

## Run order

Open `Assignment2_Student.ipynb` with the "MedAI A2 (.venv)" kernel and run all cells top to bottom
(sections 0, A, B, C, D, E, F, G, Bonus 1, Bonus 2 in that order). The notebook in the repository
contains the outputs of the final full run. Section 0.3 checks that every file in the split exists
before any real work starts.

## Report

`report/main.tex` compiles on Overleaf (upload the `report/` folder) or locally with
Tectonic, a single-file LaTeX engine:

```powershell
tectonic -X compile report\main.tex --outdir report
```

The output is `report/main.pdf`.

## Hardware

| Part | Model |
|------|-------|
| GPU  | NVIDIA GeForce RTX 5060 Ti, 16 GB |
| CPU  | AMD Ryzen 5 3600, 6 cores / 12 threads |
| RAM  | 32 GB |
| OS   | Windows 11 Pro |

## Reproducibility

- Random seed: `42` (`SEED` in Section 0.1, applied by `set_seed()`)
- cuDNN autotuning is on for speed, so results can differ slightly between runs.

## Runtime

Measured on the hardware above:

| Step | Time |
|------|------|
| Section C preprocessing (first run only, results cached in `cache/`, 13 GB) | about 15 min |
| One training run (30 epochs x 500 steps, batch 8, 64^3 patches) | about 58 min (1.9 min per epoch incl. validation) |
| Sections E, F, G (validation and test evaluation, figures) | about 13 min |
| Bonus 1 and 2 | about 9 min |
| Whole notebook with the cache already built (final run) | 2 h 15 min |

The training cases are held in RAM (about 9 GB) during Section D.
