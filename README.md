
# Sentinel-2 Satellite Data Downloader

A Python pipeline for searching, downloading, and processing Sentinel-2 Level-2A satellite imagery from the Microsoft Planetary Computer. The script supports configurable geographic bounding boxes, date ranges, cloud-cover filtering, download progress bars, and 20-meter raster resampling.

## Features

- Searches Sentinel-2 Level-2A imagery using the STAC API.
- Filters imagery by geographic extent, date range, and cloud cover.
- Downloads Sentinel-2 bands B02, B03, B04, B08, and B11.
- Displays progress bars during band downloads.
- Resamples downloaded imagery to a target resolution of 20 meters.
- Skips bands that have already been downloaded.
- Logs download activity and skipped files.
- Uses Dask for parallel scene processing.

## Requirements

- Python 3.9 or later
- Internet connection
- Sufficient disk space for satellite imagery

## Installation

Clone the repository:

```bash
git clone https://github.com/USERNAME/REPOSITORY.git
cd REPOSITORY
```

Install the required dependencies:

```bash
pip install planetary-computer pystac-client rasterio requests tqdm "dask[distributed]"
```

## Configuration

Open `download_sentinel2.py` and configure the following parameters.

| Parameter | Description |
|---|---|
| `OUTPUT_DIR` | Directory where downloaded imagery will be stored |
| `BANDS` | Sentinel-2 spectral bands to download |
| `TARGET_RES` | Target raster resolution in meters |
| `START_DATE` | Beginning of the imagery search period |
| `END_DATE` | End of the imagery search period |
| `MAX_CLOUD_COVER` | Maximum permitted scene cloud cover percentage |
| `ROI_BBOX` | Geographic bounding box |
| `WORKERS` | Number of Dask worker processes |
| `THREADS` | Threads assigned to each worker |
| `MEMLIMIT` | Memory limit per worker |

### Geographic coordinates

Configure `ROI_BBOX` using the following format:

```python
ROI_BBOX = [
    WEST_COORDINATE,
    SOUTH_COORDINATE,
    EAST_COORDINATE,
    NORTH_COORDINATE,
]
```

Replace the coordinate placeholders with numeric longitude and latitude values. The coordinate order is west, south, east, north.

### Date range

Use the ISO date format:

```python
START_DATE = "YYYY-MM-DD"
END_DATE = "YYYY-MM-DD"
```

Replace these values with the desired beginning and ending dates.

## Usage

Run the downloader:

```bash
python download_sentinel2.py
```

The script searches the STAC catalog, identifies available scenes, downloads the selected spectral bands, and resamples them to the target resolution.

Progress bars display the download status of individual bands. Logging output records completed downloads, skipped files, and errors.

## Output Structure

Downloaded imagery is organized by Sentinel-2 scene identifier.

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

The script also creates a `logs/` directory containing run logs and skipped-band logs.

## Spectral Bands

| Band | Description |
|---|---|
| B02 | Blue |
| B03 | Green |
| B04 | Red |
| B08 | Near-infrared |
| B11 | Shortwave infrared |

These bands can support downstream spectral-index calculations and satellite-based environmental analysis.

## Data Source

Imagery is accessed through the [Microsoft Planetary Computer](https://planetarycomputer.microsoft.com/) using the Sentinel-2 Level-2A STAC collection.

## Notes

- The script skips existing output bands to reduce redundant downloads.
- Download progress bars track transferred bytes; raster resampling occurs after downloading.
- Actual output resolution and alignment depend on the source raster's spatial resolution and coordinate system.
- Dask worker and memory settings should be adjusted to match available hardware.
- Download availability depends on the selected region, date range, and cloud-cover threshold.

