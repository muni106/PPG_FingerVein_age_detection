# PPG + FingerVein — Age Detection

Deep Learning university project: **estimating a subject's age (regression)** from two
biometric modalities — **FingerVein** images and **PPG** signals — and from their fusion.
See the proposal (`../proposal_helpers/proposta-rivista.md`) for the full plan.

This repository currently contains the **data-exploration / visualization** step only.

## Datasets

The raw datasets are **not** committed (they are large). They live next to this project in
`../proposal_helpers/proj_files/` as zip archives, and the notebooks read them directly:

| Modality   | Archive                                                              | Format            | Labels |
|------------|---------------------------------------------------------------------|-------------------|--------|
| FingerVein | `MMCBNU_6000.zip`                                                    | BMP images        | ⚠️ no age in the public package |
| PPG        | `brno-university-of-technology-smartphone-ppg-database-but-ppg-2.0.0.zip` | WFDB (`.hea`/`.dat`) + `subject-info.csv` | Age, Gender |

- **MMCBNU_6000** — 6000 near-infrared finger-vein images: 100 subjects × 6 fingers × 10 captures.
  It was released for finger-vein *recognition*; the bundled files do **not** include per-subject age,
  so we still need age annotations before training the FingerVein age model.
- **BUT PPG** — 10 s smartphone PPG segments at 30 Hz, stored in PhysioNet WFDB format, with a
  `subject-info.csv` providing **Age** (our regression target) and **Gender**.

## Setup

This project uses [**uv**](https://docs.astral.sh/uv/) as the package manager (Python 3.11).

```bash
uv sync          # create .venv and install dependencies
```

Open the folder in **VS Code**, install the *Python* + *Jupyter* extensions, and select the
`.venv` interpreter (`.vscode/settings.json` already points to it). Then open a notebook and run.

To run a notebook from the terminal:

```bash
uv run jupyter lab          # or: uv run jupyter notebook
```

## Notebooks

| Notebook                                   | Purpose                                                        |
|--------------------------------------------|---------------------------------------------------------------|
| `notebooks/01_fingervein_exploration.ipynb`| Browse FingerVein images, dataset structure, pixel statistics |
| `notebooks/02_ppg_exploration.ipynb`       | Subject info, age/gender distributions, raw & filtered PPG     |

## Layout

```
.
├── notebooks/         # exploration / visualization notebooks
├── data/              # local datasets (gitignored) — optional, archives are read in place
├── pyproject.toml     # uv project + dependencies
└── .vscode/           # VS Code interpreter settings
```
