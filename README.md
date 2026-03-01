# Gymnocladus Leaf Area Analysis

An R-based pipeline for measuring leaf area from flatbed scanner images and calculating Specific Leaf Area (SLA). Developed for *Gymnocladus dioicus* (Kentucky coffeetree) at the Chicago Botanic Garden, but applicable to any flatbed scan with plant material on a dark background.

## What this pipeline does

Each scan contains multiple individual leaf pieces (leaflets, rachis segments) laid on a black background. The pipeline:

1. Segments each piece from the background using Otsu's automatic thresholding
2. Numbers each detected piece and saves a labeled review image
3. Measures area per piece in cm²
4. Compiles results into a master table
5. Joins area data with oven-dry mass to calculate SLA (cm²/g)

## Requirements

- R ≥ 4.1
- RStudio (recommended)
- `EBImage` (Bioconductor) — installed automatically by `01_setup_and_calibration.Rmd`
- `tidyverse`, `fs` — installed automatically

## Scan requirements

| Property | Expected value |
|----------|---------------|
| Background color | Black |
| File format | TIFF (`.tif`) |
| Resolution | 300 DPI (any consistent DPI works — update the `dpi` parameter) |
| Scale bar | White ruler in a consistent position (bottom-right) |
| Color standard | Color calibration card (should be in the same region as the ruler) |

Leaf pieces must be clearly **separated** on the scanner bed. Touching pieces will be merged into a single labeled object and cannot be automatically split.

## Workflow

Run the three notebooks in order:

### 1. `01_setup_and_calibration.Rmd`

- Installs `EBImage` and dependencies
- Set your `scan_folder` path here
- Confirms scanner resolution and scale bar calibration
- Runs a test segmentation on one image so you can verify the pipeline before batch processing

### 2. `02_batch_segment.Rmd`

- Processes all TIFF files in `scan_folder`
- For each scan, saves:
  - **`output/labeled_images/<scan_id>_labeled.png`** — original image with each piece outlined in red and numbered
  - **`output/area_tables/<scan_id>_areas.csv`** — one row per piece
- Compiles everything into **`output/area_tables/master_leaf_areas.csv`**

After this runs, **review the labeled PNGs** and note any piece numbers that represent non-leaf objects (e.g., fragments of the color card). Enter those as exclusions in notebook 3.

### 3. `03_sla_calculation.Rmd`

- Loads the master area table
- Applies any manual piece exclusions you specify
- Joins area data with your dry weight CSV
- Calculates SLA and exports `output/area_tables/sla_results.csv`

## Output files

| File | Description |
|------|-------------|
| `output/area_tables/master_leaf_areas.csv` | One row per piece, all scans |
| `output/area_tables/area_per_individual.csv` | Total area summed per individual |
| `output/area_tables/sla_results.csv` | Final SLA values (cm²/g = m²/kg) |
| `output/labeled_images/*_labeled.png` | Review images with numbered pieces |

### master_leaf_areas.csv column reference

| Column | Units | Description |
|--------|-------|-------------|
| `scan_id` | — | Filename without `.tif` |
| `individual_id` | — | Scan ID with trailing scan number stripped |
| `piece_num` | — | Sequential piece number within this scan |
| `orig_label` | — | Raw EBImage label ID (for debugging) |
| `area_px` | pixels | Piece area in pixels |
| `area_mm2` | mm² | Piece area in mm² |
| `area_cm2` | cm² | Piece area in cm² |
| `centroid_x` | px | Horizontal centroid (from left edge) |
| `centroid_y` | px | Vertical centroid (from top edge) |
| `elongation` | — | Longest/shortest radius ratio (diagnostic) |

## Key parameters

All adjustable in the `user-config` chunk at the top of each notebook:

| Parameter | Default | Description |
|-----------|---------|-------------|
| `dpi` | 300 | Scanner resolution in dots per inch |
| `min_area_cm2` | 0.25 | Minimum piece area to keep; removes dust and debris |
| `max_elongation` | 6.0 | Maximum elongation ratio; removes the ruler automatically |
| `review_scale` | 0.25 | Resolution of labeled review PNGs (fraction of original) |

## Adapting to other species or scanners

- **Different DPI**: update `dpi` in the config chunk; all conversions are derived from it
- **Different background**: if the background is not pure black, adjust the Otsu offset or switch to a manual threshold in `02_batch_segment.Rmd`
- **Different scan layout**: if the ruler is in a different position, the elongation filter (`max_elongation`) will still remove it automatically — you don't need to change any spatial parameters
- **Overlapping pieces**: watershed segmentation can be added to `segment_scan()` using `EBImage::watershed()` if needed

## Citation / contact

Developed by Natalie Love, Chicago Botanic Garden.
For questions open an issue on this repository.
