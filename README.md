# EEG Analysis Workflow for CZD Data (Intern 1)

This repository contains a reproducible EEG analysis workflow for CZD EDF recordings, focused on the work completed by Intern 1.

The pipeline covers:
- loading EDF metadata without full preload,
- inspecting the recording inventory,
- plotting raw EEG traces and PSDs,
- applying basic preprocessing (notch + band-pass filtering),
- building a bipolar montage,
- summarizing channel quality,
- scaling the analysis to a large EDF file using cropped windows,
- exporting annotations and optional event markers.

## Repository contents

- `analiza_eeg.ipynb` — workflow for the smaller CZD EDF file.
- `analiza_eeg_duzy.ipynb` — low-memory workflow for a large EDF file using cropped windows.
- `inventory_table.csv` — example inventory export.
- `figures/` — figures generated from the smaller-file workflow.
- `figures2/` — figures and exports generated from the large-file workflow.

## Environment setup

Create and activate a Python environment:

```bash
conda create -n eeg python=3.11 -y
conda activate eeg
pip install mne pyedflib numpy scipy pandas matplotlib jupyterlab yasa
```

If `pip` refuses on Linux, add `--break-system-packages`.

Optional tools:
- EDFbrowser for visual inspection of EDF files.
- `mdbtools` for reading Compumedics `.mdb` event files:

```bash
sudo apt install mdbtools
```

Sanity check:

```bash
python -c "import mne; print(mne.__version__)"
```

## Workflow overview

### 1. Small-file inspection

Use `analiza_eeg.ipynb` for the first pass on the smaller CZD EDF file.

Steps include:
1. Read the EDF header with `mne.io.read_raw_edf(..., preload=False)`.
2. Save an inventory table with:
   - number of channels,
   - channel names,
   - sampling frequency,
   - duration,
   - start time,
   - units.
3. Crop a short window (e.g. 30 s), load it into memory, and plot a subset of channels.
4. Apply a notch filter at 50 Hz and harmonics, plus a band-pass filter.
5. Compute and inspect PSD.
6. Build a bipolar montage within electrodes.
7. Create a per-channel summary table for flat/railed/noisy channels.
8. Save clean figures.

### 2. Large-file workflow

Use `analiza_eeg_duzy.ipynb` for the larger EDF file.

This version is designed for very large recordings and uses a low-memory strategy:
- process only cropped windows (for example: start, middle, end),
- keep `preload=False`,
- read only small data fragments at a time,
- extract annotations from EDF headers if present,
- optionally try to export events from `EVENTS.MDB` using `mdb-export`,
- generate a spectrogram for one window.

## Notes

- EDF annotations are normal and may appear as an extra channel; they are often useful as event markers.
- `EVENTS.MDB` is optional and is not created by this workflow automatically.
- For very large files, avoid loading the full recording into memory in one step.
- The large-file workflow is intentionally conservative and should be run as a test first before scaling further.

## Expected outputs

Depending on the notebook run, the workflow produces:
- CSV inventories and summaries,
- raw trace plots,
- PSD plots,
- bipolar montage plots,
- spectrogram images,
- event/annotation exports.
