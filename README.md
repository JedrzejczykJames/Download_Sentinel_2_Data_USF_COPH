# Sentinel-2 Satellite Data Downloading and Tile Stitching

A Python pipeline for searching, downloading, and processing Sentinel-2 Level-2A satellite imagery from the Microsoft Planetary Computer. The workflow downloads selected spectral bands, resamples imagery to a target resolution, and stitches downloaded tiles into georeferenced GeoTIFF mosaics for downstream remote-sensing and environmental analysis.

## Features

- Searches Sentinel-2 Level-2A imagery through the STAC API.
- Filters imagery by geographic extent, date range, and cloud-cover percentage.
- Downloads Sentinel-2 spectral bands B02, B03, B04, B08, and B11.
- Displays progress bars during downloads and raster processing.
- Resamples downloaded imagery to a target resolution of 20 meters.
- Skips bands that have already been downloaded.
- Logs download activity, skipped files, and processing results.
- Uses Dask for parallel scene processing.
- Stitches multiple raster tiles into larger geographic mosaics.
- Uses chunk-based processing to help manage memory usage.
- Calculates median pixel values where input rasters overlap.
- Produces georeferenced GeoTIFF outputs in EPSG:4326.
- Supports configurable worker counts, memory limits, chunk sizes, and reflectance scaling.

## Requirements

- Python 3.9 or later
- Internet connection for downloading imagery
- Sufficient disk space for satellite imagery and intermediate rasters
- GDAL command-line utilities for the optional GDAL-based mosaic workflow

## Installation

Clone the repository:

```bash
git clone https://github.com/USERNAME/REPOSITORY.git
cd REPOSITORY
```

Install the required Python dependencies:

```bash
pip install planetary-computer pystac-client rasterio requests tqdm "dask[distributed]" numpy
```

The stitching script can use GDAL utilities for mosaic assembly when available. Install GDAL separately if you want to use `gdalbuildvrt` and `gdal_translate`. Rasterio-based merging is used as a fallback when the required GDAL command-line tools are unavailable.

## Scripts

| Script | Purpose |
|---|---|
| `download_sentinel2.py` | Searches for Sentinel-2 scenes and downloads selected spectral bands. |
| `median_composite.py` | Processes downloaded tiles and creates stitched, georeferenced band mosaics. |

## Configuration

### 1. Downloading Sentinel-2 imagery

Open `download_sentinel2.py` and configure the following parameters.

| Parameter | Description |
|---|---|
| `OUTPUT_DIR` | Directory where downloaded imagery will be stored |
| `BANDS` | Sentinel-2 spectral bands to download |
| `TARGET_RES` | Target raster resolution in meters |
| `START_DATE` | Beginning of the imagery search period |
| `END_DATE` | End of the imagery search period |
| `MAX_CLOUD_COVER` | Maximum permitted scene cloud-cover percentage |
| `ROI_BBOX` | Geographic bounding box |
| `WORKERS` | Number of Dask worker processes |
| `THREADS` | Threads assigned to each worker |
| `MEMLIMIT` | Memory limit per worker |

#### Geographic coordinates

Configure `ROI_BBOX` using the following format:

```python
ROI_BBOX = [
    WEST_COORDINATE,
    SOUTH_COORDINATE,
    EAST_COORDINATE,
    NORTH_COORDINATE,
]
```

Replace the placeholders with numeric longitude and latitude values. The coordinate order is west, south, east, north.

#### Date range

Use the ISO date format:

```python
START_DATE = "YYYY-MM-DD"
END_DATE = "YYYY-MM-DD"
```

Replace these values with the desired beginning and ending dates.

### 2. Stitching and compositing tiles

Open `median_composite.py` and configure the input and output directories and processing parameters.

| Parameter | Description |
|---|---|
| `--tiles_dir` | Directory containing subdirectories of downloaded Sentinel-2 tiles |
| `--out_dir` | Directory where the final stitched band mosaics will be written |
| `--workers` | Number of parallel processing workers |
| `--memory_limit` | Memory limit assigned to each worker |
| `--chunk_size` | Raster chunk size used during processing |
| `--nodata` | No-data value used for output rasters |
| `--scale` | Reflectance scaling factor, normally `10000` |
| `--keep_chunks` | Retains intermediate chunk files when enabled |

The stitching script expects the input directory to contain subdirectories with the required band files. Each scene or tile directory should contain the corresponding GeoTIFFs for B02, B03, B04, B08, and B11.

## Usage

### Step 1: Download imagery

Run the downloader:

```bash
python download_sentinel2.py
```

The script searches the STAC catalog, identifies available scenes, downloads the selected spectral bands, and resamples them to the configured target resolution.

### Step 2: Stitch and composite tiles

Run the stitching script:

```bash
python median_composite.py --tiles_dir /INSERT/FILE/HERE --out_dir /INSERT/FILE/HERE
```

Replace `/INSERT/FILE/HERE` with the appropriate input and output directory paths.

Optional processing parameters can be specified through command-line arguments:

```bash
python median_composite.py \
    --tiles_dir /INSERT/FILE/HERE \
    --out_dir /INSERT/FILE/HERE \
    --workers 4 \
    --memory_limit 12GB \
    --chunk_size 4096 \
    --nodata 0 \
    --scale 10000
```

Use `--keep_chunks` if you want to retain intermediate chunk files for inspection or additional processing.

The script processes raster chunks in parallel, reports progress, and assembles the processed chunks into final mosaics for each spectral band.

## Output Structure

### Downloaded imagery

```text
OUTPUT_DIR/
├── SENTINEL_SCENE_ID/
│   ├── B02_20m.tif
│   ├── B03_20m.tif
│   ├── B04_20m.tif
│   ├── B08_20m.tif
│   └── B11_20m.tif
└── ...
```

The downloader also creates logging output for recording completed downloads, skipped bands, and errors.

### Stitched mosaics

```text
OUTPUT_DIR/
├── B02.tif
├── B03.tif
├── B04.tif
├── B08.tif
└── B11.tif
```

Intermediate chunk files may be stored in a temporary `_chunks/` directory during processing. These are removed after successful assembly unless `--keep_chunks` is enabled.

The final GeoTIFFs are georeferenced rasters in EPSG:4326 and can be used in GIS software and downstream raster-processing workflows.

## Spectral Bands

| Band | Description | Common applications |
|---|---|---|
| B02 | Blue | Water and atmospheric features |
| B03 | Green | Vegetation and water analysis |
| B04 | Red | Vegetation indices and land-cover analysis |
| B08 | Near-infrared | Vegetation condition and biomass-related indices |
| B11 | Shortwave infrared | Moisture-sensitive and land-cover indices |

These bands support downstream calculation of spectral indices, including NDVI, NDWI, MNDVI, MNDWI, SAVI, and SABI, when the required band combinations and index definitions are used.

## Processing Overview

1. **Scene search:** The downloader queries the Sentinel-2 Level-2A STAC collection using the configured region, date range, and cloud-cover threshold.
2. **Band download:** Selected spectral bands are downloaded and resampled to the target resolution.
3. **Raster preparation:** The stitching script reads the downloaded tiles and establishes a common geographic output grid.
4. **Chunk processing:** The raster extent is divided into chunks to reduce memory requirements during reprojection and compositing.
5. **Median compositing:** Median values are calculated for overlapping input rasters at each pixel, with reflectance scaling and no-data handling applied.
6. **Mosaic assembly:** Processed chunks are combined into final GeoTIFFs for each spectral band.

**Important:** Median compositing is most meaningful when overlapping input rasters represent multiple observations of the same locations, such as scenes from different dates. When inputs are primarily adjacent geographic tiles, the process functions mainly as a stitching and mosaicking workflow, with median calculations applying only where their footprints overlap.

## Data Source

Imagery is accessed through the [Microsoft Planetary Computer](https://planetarycomputer.microsoft.com/) using the Sentinel-2 Level-2A STAC collection.

## Notes

- Existing downloaded bands are skipped to reduce redundant downloads.
- Cloud-cover filtering applies to scene-level metadata and does not guarantee that every pixel is cloud-free.
- The downloader's target resolution does not increase the native spatial detail of a source band.
- The stitching workflow uses EPSG:4326 and an approximate degree-based resolution corresponding to 20 meters at the equator. Ground resolution therefore varies with latitude.
- The default scale factor of `10000` is intended for Sentinel-2 surface-reflectance values stored as scaled integers. Confirm the source data's scaling before applying it.
- The configured no-data value should be consistent with the source rasters and downstream processing requirements.
- Worker count, chunk size, and memory limits should be adjusted to match available hardware.
- Final mosaics contain spectral bands, not automatically calculated spectral indices.
- Imagery availability depends on the selected geographic region, date range, and cloud-cover threshold.
