# EEG Analysis Workflow for CZD Data

This repository contains a reproducible EEG analysis workflow for CZD EDF recordings. The current focus is to inspect the recording, build a bipolar montage, summarize channel quality, and explore candidate seizure-onset zone (SOZ) channels.

## Overview

The workflow covers:
- loading EDF metadata without fully preloading the file,
- inspecting the recording inventory,
- plotting raw EEG traces and power spectra,
- applying basic preprocessing (notch and band-pass filtering),
- building bipolar channels from adjacent electrodes,
- summarizing channel quality,
- processing large recordings with cropped windows,
- exporting annotations and optional event markers.

## Repository contents

- [analiza_eeg.ipynb](analiza_eeg.ipynb) — first-pass workflow for the smaller CZD EDF file.
- [bipolar_pipeline.ipynb](bipolar_pipeline.ipynb) — updated low-memory workflow for the larger EDF file.
- [inventory_table.csv](inventory_table.csv) — example inventory export.
- [tables](tables) — exported CSV summaries and annotation tables.
- [figures](figures) — figures generated from the analysis workflow.

## Environment setup

Create and activate a Python environment:

```bash
conda create -n eeg python=3.11 -y
conda activate eeg
pip install mne pyedflib numpy scipy pandas matplotlib jupyterlab yasa
```

Optional tools:
- EDFbrowser for visual inspection of EDF files.
- mdbtools for reading Compumedics .mdb event files:

```bash
sudo apt install mdbtools
```

Sanity check:

```bash
python -c "import mne; print(mne.__version__)"
```

## Workflow overview

### 1. Small-file inspection

Use [analisys_short_file.ipynb](analisys_short_file.ipynb) for the first pass on the smaller CZD EDF file.

Steps include:
1. Read the EDF header with `mne.io.read_raw_edf(..., preload=False)` and remember to change paths accordingly.
2. Save an inventory table with:
   - number of channels,
   - channel names,
   - sampling frequency,
   - duration,
   - start time,
   - units.
3. Crop a short window (for example 30 s), load it into memory, and plot a subset of channels.
4. Apply a notch filter at 50 Hz and harmonics, plus a band-pass filter.
5. Compute and inspect power spectral density (PSD).
6. Build a bipolar montage within electrodes.
7. Create a per-channel summary table for flat, railed, and noisy channels.
8. Save figures and tables.

### 2. Updated large-file workflow

Use [analisys_long_file.ipynb](analisys_long_file.ipynb) for the larger EDF file.

This version is designed for only one large recordings and uses a low-memory strategy:
- process only cropped windows (currently start, middle, and end),
- keep `preload=False`,
- read only small data fragments at a time,
- build bipolar channels from neighboring electrodes and filter,
- generate summaries for channel quality and signal behavior,
- export outputs for later review.

What is happening in the updated bipolar pipeline:
- the EDF is opened in a memory-safe way,
- analysis windows are defined across the recording,
- a subset of data is loaded for inspection,
- the bipolar montage is created from adjacent contacts with filtration,
- plots and summaries are generated for each channel or channel pair,
- annotations are exported for review and possible SOZ analysis.

## Current questions and notes

These are the main points still under discussion:

- Annotation durations: many annotation rows currently show a duration of `0.0` seconds. This should be clarified. Is this expected from the export format, or are the original events supposed to have a non-zero duration?
- Noise assessment: what should a noisy channel look like? 
- Channel quality: the current workflow assumes that all channels are acceptable, but this should be verified. Are there any channels that look suspicious even if they are not marked as noisy?
- Parameters: the exact preprocessing parameters should be documented, including filter settings, window length, and any thresholding rules used in the QC step.
- Units and scaling: confirm whether the EDF data are stored in microvolts or another unit, and whether any unit conversion or scaling is needed.
- Duration mismatch: compare the duration reported in the EDF header with the duration implied by the annotations and the recording timeline. Differences may reflect gaps, truncation, or missing segments.
- channels such as `RM 6` to `RM 9` we eliminatied

## Known issues / observations

- Filtering did not strongly change the output, which suggests the signal is already quite clean.
- Noisy channels are not yet clearly labeled in the exported outputs.
- `RIP6-RIP9` is not a valid bipolar channel in this workflow because the difference is larger than 1 contact apart.
- On some systems, saving figures can fail depending on the backend or environment.

## Expected outputs

Depending on the notebook run, the workflow produces:
- CSV inventories and summaries,
- raw trace plots,
- PSD plots,
- bipolar montage plots,
- spectrogram images,
- annotation exports and review tables.

