# EO_DISTANCE_LAND_SEAPORT

Module for computing **NDVI** (Normalised Difference Vegetation Index) over land pixels in the vicinity of the Port of Gdańsk, using Sentinel-2 L2A imagery. It generates monthly mean NDVI rasters and extracts index values at a fixed set of control points in the Stogi area (pine forest belt adjacent to the port).

Formula: `NDVI = (B8 − B4) / (B8 + B4)`

---

## Source files

| File | Role |
|---|---|
| `config_ndvi.py` | All parameters: AOI, year, month, control points, paths, thresholds |
| `run_land_component.py` | Main runner – computes NDVI rasters and triggers point extraction + plotting |
| `run_ndvi_year.py` | Year and month range config (edit manually before standalone extraction) |
| `plot_ndvi_points_year.py` | Generates per-month PNG plots from extracted point CSVs |

---

## How to run

### Option 1 – CLI arguments (recommended)

```bash
cd EO_DISTANCE_LAND_SEAPORT
python run_land_component.py --year 2024 --start-month 1 --end-month 12
```

Add `--force` to overwrite existing output files.

### Option 2 – JSON context file

```bash
python run_land_component.py --context /path/to/context.json
```

`context.json` format:
```json
{
  "year": 2024,
  "months": [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12],
  "paths": {
    "land_preprocessed": "../EO_ENVIRONMENTAL_PREPROCESSING/preprocessed/land"
  }
}
```

### Option 3 – plot-only (rasters and point CSVs already exist)

Edit `run_ndvi_year.py` (set `YEAR` and `MONTHS`), then run:

```bash
python plot_ndvi_points_year.py
```

---

## Input data

Reads monthly per-band mean rasters (`b04_mean`, `b08_mean`) produced by `EO_ENVIRONMENTAL_PREPROCESSING`. Default path:

```
../EO_ENVIRONMENTAL_PREPROCESSING/preprocessed/land/<year>_<MM>/
```

Water mask (for filtering out water pixels):
```
../EO_ENVIRONMENTAL_PREPROCESSING/assets/maska_wody/warstwa_wody_terra_checked.tif
```

AOI: circular area with **10 km radius** centred on the Port of Gdańsk (`lat 54.4008, lon 18.6992`). Resolution: **10 m**.

---

## Output structure

```
outputs/
└── ndvi_month_out/
    ├── <year>_<MM>/
    │   ├── ndvi_mean_<year>_<MM>.tif          # float32 NDVI raster
    │   ├── ndvi_mean_<year>_<MM>_rgb.tif      # RGB visualisation
    │   ├── ndvi_points_<year>_<MM>.csv        # values at Stogi control points
    │   ├── monthly_stats.csv                  # min/max/mean/valid_px for the month
    │   └── inventory.csv                      # Sentinel scenes used for compositing
    ├── ndvi_points_<year>_all_months.csv      # all points, full year combined
    ├── plot_summary_<year>.csv                # plot generation status log
    └── plots_points_<year>/
        └── ndvi_points_STOGI_<year>_<MM>.png  # NDVI profile plot across Stogi points
```

---

## Control points

5 points in the Stogi pine forest along the coastline east of the Gdańsk port basin. Each point uses a **30 m** buffer radius for pixel aggregation.

| ID | Name | Lat | Lon |
|---|---|---|---|
| 1 | Stogi 1 Pine | 54.3808 | 18.7120 |
| 2 | Stogi 2 Pine | 54.3776 | 18.7189 |
| 3 | Stogi 3 Pine | 54.3703 | 18.7362 |
| 4 | Stogi 4 Pine | 54.3675 | 18.7481 |
| 5 | Stogi 5 Pine | 54.3644 | 18.7616 |

---

## Dependencies

Requires `EO_ENVIRONMENTAL_PREPROCESSING` to have been run first — the `preprocessed/land/` folder with per-month band rasters (`b04_mean`, `b08_mean`) must exist.
