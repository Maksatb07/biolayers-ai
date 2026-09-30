# Handoff to the Software team: overlay and per-cell data format

This folder holds real pipeline output in the exact format `src/demo.py`
writes, so the frontend can be built against it. Regenerate any of it with
the commands at the bottom.

Research and educational system. Not a clinical diagnostic. Any UI that shows
these outputs must keep the evidence level visible next to each statement.

## Contents

| Folder | Source | What it shows |
|---|---|---|
| `u373_seq02/` | Cell Tracking Challenge PhC-C2DH-U373, sequence 02 (115 frames) | Time-lapse: tracking, migration, phenotype labels, literature links. Contains the flagship cell `U373-02-T7`. |
| `bbbc039_sample/` | One BBBC039 image (U2OS nuclei, CC0) | Single image: segmentation, morphology, single-frame flags. |

## Time-lapse output (`u373_seq02/`)

| File | Format | Notes |
|---|---|---|
| `overlay.mp4` | 1044 x 836, 8 fps, mp4v | Ready-made overlay: outlines and trails coloured by phenotype label. 28 px header, 28 px footer. |
| `overlay_last_frame.png` | PNG | Last video frame, for thumbnails. |
| `masks/tNNN.png` | 16-bit grayscale PNG, one per frame, original resolution (696 x 520) | Pixel value = `track_id`, 0 = background. The same cell keeps the same value in every frame. Draw your own overlays from these. |
| `tracks.csv` | CSV, one row per cell per frame | `frame`, `track_id`, `centroid_row`, `centroid_col`, `area`, `perimeter`, `circularity`, `eccentricity`, `aspect_ratio`, `solidity`, intensity and texture columns. Units are pixels. |
| `cells.json` | JSON | Everything the UI needs per cell (see below). |
| `flagship.png` | PNG | The Image -> Cell -> Phenotype -> Biological Entity chain for the flagship cell. |

The raw frames are not redistributed here. Get them from the Cell Tracking
Challenge (dataset PhC-C2DH-U373) and place them under
`data/CTC/PhC-C2DH-U373/`. Frame `tNNN.tif` matches `masks/tNNN.png`.
Pixel size 0.65 um, frame interval 15 min.

## Single-image output (`bbbc039_sample/`)

| File | Format | Notes |
|---|---|---|
| `image.png` | 8-bit PNG | The input image, contrast-stretched for display. |
| `mask.png` | 16-bit grayscale PNG | Pixel value = `cell_id`, 0 = background. |
| `overlay.png` | PNG, 2x upscaled | Outlines coloured by label: green = no flag, orange = abnormal morphology, red = apoptotic/dead, grey = cut by image edge. |
| `cells.csv` | CSV, one row per cell | All measurements plus `label`, `rule`, `confidence`, `cell_prob`. |
| `cells.json` | JSON | Per cell: `cell_id`, `centroid` `[row, col]`, `contour` (list of `[x, y]` pixel points), statements. |

## `cells.json` structure

```
{
  "title", "generated", "disclaimer",
  "dataset": { "name", "cell_type", "pixel_size_um", "frame_interval_min", "frames", "caveats": [statement] },
  "evidence_levels": { "<level>": "<plain-language meaning>" },
  "model_cards": { "segmentation" | "tracking" | "phenotype_rules" | "literature_linker":
                   { "model", "version", "confidence", "dataset", "evaluation_metric", "limitations": [] } },
  "flagship_cell": "U373-02-T7",
  "cells": [ {
      "cell_id": "U373-02-T7", "sequence": "02", "track_id": 7,
      "observations": [statement],   // measured from the image
      "phenotype":    [statement],   // rule-based labels
      "literature":   [statement],   // published findings, with citations
      "hypothesis":   [statement]    // explicitly untested
  } ]
}
```

A **statement** is:

```
{
  "level": "observed_in_image" | "supported_by_literature" | "inferred_by_model",
  "statement": "human-readable sentence",
  "source": "<key into model_cards>",
  "confidence": number or null,        // meaning depends on source, see model_cards[source].confidence
  "values": { ... },                   // numeric fields, on measurement statements
  "evidence": { ... },                 // the feature values behind a rule label
  "citations": [ { "citation", "pmid", "doi" } ],   // literature statements only
  "how_to_test": "..."                 // hypothesis statements only
}
```

## Display rules (from the project brief)

1. Show the `level` with every statement. The three levels must never be
   rendered as if they were equivalent. Suggested colours: green (observed),
   blue (literature), orange (inferred).
2. For every AI-derived value, make its model card reachable: model, version,
   confidence, dataset, evaluation metric, limitations.
3. `confidence` is not a probability. Show it with the explanation in
   `model_cards[source].confidence`.
4. Keep the disclaimer visible.
5. Do not drop `hypothesis` wording such as "Hypothesis only" and "Not tested".

## Regenerate

```
python src/demo.py --sequence 02 --out handoff/u373_seq02
python src/demo.py --image data/BBBC039/images/IXMtest_A02_s1_w1051DAA7C-7042-435F-99F0-1E847D9B42CB.tif --out handoff/bbbc039_sample
```
