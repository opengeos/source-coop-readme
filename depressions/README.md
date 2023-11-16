---
jupytext:
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.15.2
kernelspec:
  display_name: geo
  language: python
  name: python3
---

# National Surface Depressions

[![image](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/opengeos/source-coop-readme/blob/main/depressions/README.ipynb)

## Description

This dataset represents the extent and location of surface depressions derived from the 10-m resolution dataset from the 3D Elevation Program ([3DEP](https://www.usgs.gov/3d-elevation-program)). The levet-set algorithm available through the [lidar](https://lidar.gishub.org) Python package was used the process the DEM and delineate surface depressions at the HU8 watershed scale.

## Reference

- Wu, Q., Lane, C. R., Wang, L., Vanderhoof, M. K., Christensen, J. R., & Liu, H. (2019). Efficient Delineation of Nested Depression Hierarchy in Digital Elevation Models for Hydrological Analysis Using Level‐Set Method. JAWRA Journal of the American Water Resources Association , 55(2), 354-368. https://doi.org/10.1111/1752-1688.12689

## Environment setup

First, create a conda environment with the required packages:

```bash
conda create -n gdal python=3.11
conda activate gdal
conda install -c conda-forge mamba
mamba install -c conda-forge libgdal-arrow-parquet gdal leafmap
pip install lonboard
```

If you are using Google Colab, you can uncomment the following to install the packages and restart the runtime after installation.

```{code-cell} ipython3
# %pip install leafmap lonboard
```

## Data access

The dataset was derived from the 10-m resolution dataset from the 3D Elevation Program ([3DEP](https://www.usgs.gov/3d-elevation-program)) at HU8 watershed scale. The results were then merged at the HU2 watershed scale. Below is a map of the National Hydrography Dataset ([NHD](https://www.usgs.gov/media/images/watershed-boundary-dataset-structure-visualization)) watershed boundary structure.

![](https://i.imgur.com/UFivxid.png)

The script below can be used to access the data using [DuckDB](https://duckdb.org). The script uses the [duckdb](https://duckdb.org) Python package.

```{code-cell} ipython3
import duckdb

con = duckdb.connect()
con.install_extension("spatial")
con.load_extension("spatial")

hu2 = "06"    # Change to the HU2 of your choice
url = f"https://data.source.coop/giswqs/depressions/all_dep/HU2_{hu2}.parquet"
con.sql(f"SELECT * EXCLUDE geometry, ST_GeomFromWKB(geometry) as geometry FROM '{url}'")
```

Find out the total number non-floodplain wetlands in the selected watershed:

```{code-cell} ipython3
con.sql(f"SELECT COUNT(*) FROM '{url}'")
```

Alternatively, you can use the aws cli to access the data directly from the S3 bucket:

```bash
aws s3 ls s3://us-west-2.opendata.source.coop/giswqs/depressions/all_dep/
```

## Data visualization

To visualize the data, you can use the [leafmap](https://leafmap.org) Python package with the [lonboard](https://github.com/developmentseed/lonboard) backend. The script below shows how to visualize the data.

```{code-cell} ipython3
import leafmap

hu2 = "06"    # Change to the HU2 of your choice
url = f"https://data.source.coop/giswqs/depressions/all_dep/HU2_{hu2}.parquet"
gdf = leafmap.read_parquet(url, return_type='gdf', src_crs='EPSG:5070', dst_crs='EPSG:4326')
# leafmap.view_vector(gdf, get_fill_color=[0, 0, 255, 128])
```

## Data analysis

The depression dataset has two variations: `all_dep` and `nfp_dep`. The `all_dep` dataset contains all depressions, while the `nfp` dataset contains non-floodplain depressions.

### All depressions

Find out the total number of surface depression in the contiguous United States (CONUS):

```{code-cell} ipython3
con.sql(f"""
SELECT COUNT(*) AS Count
FROM 's3://us-west-2.opendata.source.coop/giswqs/depressions/all_dep/*.parquet'
""")
```

Calculate some descriptive statistics of all surface depressions:

```{code-cell} ipython3
con.sql(f"""
SELECT sum(area) as total_area, median(area) AS median_area, median(volume) AS median_volume, median(avg_depth) AS median_depth
FROM 's3://us-west-2.opendata.source.coop/giswqs/depressions/all_dep/*.parquet'
""")
```

### Non-floodplain depressions

Find out the total number non-floodplain depressions in the contiguous United States (CONUS):

```{code-cell} ipython3
con.sql(f"""
SELECT COUNT(*) AS Count
FROM 's3://us-west-2.opendata.source.coop/giswqs/depressions/nfp_dep/*.parquet'
""")
```

Calculate some descriptive statistics of non-floodplain surface depressions:

```{code-cell} ipython3
con.sql(f"""
SELECT sum(area) as total_area, median(area) AS median_area, median(volume) AS median_volume, median(avg_depth) AS median_depth
FROM 's3://us-west-2.opendata.source.coop/giswqs/depressions/nfp_dep/*.parquet'
""")
```
