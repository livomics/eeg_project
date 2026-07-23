# EEG Analysis Workflow for CZD Data

This repository contains a reproducible EEG analysis workflow for CZD EDF recordings. The project focuses on automated preprocessing, seizure-window extraction, bipolar montage generation, spectral analysis, and visualization to support seizure onset zone (SOZ) exploration.

## Overview

This repository contains two complementary EEG analysis workflows for CZD EDF recordings:

- `analisys_short_file.ipynb` for initial inspection of a shorter recording, including metadata review, bipolar montage generation, inspection of eeg trace signals and PSD before and after filtering,, and basic channel-quality assessment;
- `SOZ_analisys.ipynb` implements an automated seizure-analysis workflow for multiple EDF recordings. It extracts annotations, detects seizure events (`NAPAD`), creates seizure-centered windows, applies signal preprocessing, constructs a bipolar montage, computes PSD, detects disconnected channels, and exports per-recording and per-seizure results.

The workflow is designed to support reproducible exploration of SEEG recordings and candidate seizure onset zone (SOZ) channels while keeping large EDF datasets outside the Git repository by using a local config.py configuration.

## Repository contents

- [SOZ_analisys.ipynb](SOZ_analisys.ipynb) — main notebook implementing the complete seizure-focused analysis workflow.
- [analisys_short_file.ipynb](analisys_short_file.ipynb) — exploratory notebook for the initial inspection of EDF recordings.
- [config.example.py](config.example.py) — example local configuration file defining the EDF data directory.
- [inventory_table.csv](inventory_table.csv) — example inventory export.
- [tables](tables) — exported CSV summaries and annotation tables.
- [figures](figures) — figures generated during the analysis workflow.
- [SOZ](SOZ) — per-file and per-seizure output folders created by the SOZ analysis notebook.

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

change the [config.example.py](config.example.py) to config.py file based on your edf files location, do this once - it will not be added to git

### 1. Short-file inspection

Use [analisys_short_file.ipynb](analisys_short_file.ipynb) for the initial inspection of a shorter EDF recording.

Steps include:

1. Read the EDF file with `mne.io.read_raw_edf(..., preload=False)` using the data path defined in the local configuration.
2. Save an inventory table containing:
   - number of channels,
   - channel names,
   - sampling frequency,
   - recording duration,
   - start time,
   - signal units.
3. Crop a short window, currently 90 seconds, load it into memory.
4. Build a bipolar montage using adjacent contacts from the same electrode. Save the trace EEG
5. Compute and save the PSD of the unfiltered bipolar signal.
6. Apply notch filtering at 50 and it's harmonics and apply the selected frequency filter - for now 1-250Hz.
7. Plot the filtered signal and compute its PSD for comparison with the unfiltered data.
8. Compare selected channels and review activity in standard frequency bands.
9. Create a per-channel summary table for flat, saturated, and high-noise channels.
10. Save figures and channel-quality results.

### 2. SOZ-focused seizure analysis

Use [SOZ_analisys.ipynb](SOZ_analisys.ipynb) for automated seizure-focused analysis of multiple EDF recordings.

This notebook:

1. Loads all EDF files from the directory specified in `config.py`.
2. Reads EDF annotations and exports them as per-recording CSV inventories.
3. Detects seizure events whose descriptions contain `NAPAD`.
4. Extracts a 90-second analysis window for each seizure (60 seconds before onset and 30 seconds after onset).
5. Applies notch filtering (50Hz and it's harmonics) followed by a 1–250 Hz band-pass filter.
6. Creates a bipolar montage by pairing adjacent contacts from the same electrode.
7. Computes the Power Spectral Density (PSD) for all bipolar channels.
8. Automatically detects disconnected or flat channels based on PSD and excludes them from further spectral analysis.
9. Exports PSD figures, annotation tables, and per-seizure analysis results to the [SOZ](SOZ) directory.
10. Takes a spectogram of every channel for every seizure.

## Current questions and notes

These are the main points still under discussion:

- Annotation durations: many annotation rows currently show a duration of `0.0` seconds. This should be clarified. Is this expected from the export format, or are the original events supposed to have a non-zero duration?
- Noise assessment: what should a noisy channel look like? 
- Channel quality: the current workflow assumes that all channels are acceptable, but this should be verified. Are there any channels that look suspicious even if they are not marked as noisy?
- Parameters: the exact preprocessing parameters should be documented, including filter settings, window length, and any thresholding rules used in the QC step.
- Units and scaling: confirm whether the EDF data are stored in microvolts or another unit, and whether any unit conversion or scaling is needed.
- Duration mismatch: compare the duration reported in the EDF header with the duration implied by the annotations and the recording timeline. Differences may reflect gaps, truncation, or missing segments.
- channels such as `RM 6` to `RM 9` we eliminatied

### Channel Filtering & Quality Control

During seizures, physical movement can cause some electrodes to lose contact or fall off completely. These disconnected channels record no physiological activity (resulting in flatlines or numerical zeros) and are not useful for localization or imaging. 

To maintain data integrity:
* The processing pipeline automatically detects these flat/invalid channels by scanning for near-zero power in the Power Spectral Density (PSD) analysis.
* These broken channels are dynamically filtered out and excluded from subsequent steps, such as spectrogram generation, to save computation time and keep the output clean.
* While the code can be modified to log the specific names of these dropped channels, this information is treated as redundant for the primary localization objectives of this analysis.

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

