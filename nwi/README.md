---
jupytext:
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.15.2
---

# National Wetlands Inventory

[![image](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/opengeos/source-coop-readme/blob/main/nwi/README.ipynb)

## Description

This dataset is a copy of the [National Wetlands Inventory](https://www.fws.gov/program/national-wetlands-inventory), offering the data in more GIS-friendly and [cloud-native geospatial](https://cloudnativegeo.org) formats. The original dataset is distributed as zipped Geodatabase files, and is available for download from [here](https://www.fws.gov/program/national-wetlands-inventory/download-state-wetlands-data).

## Data download

The script below was used to download the data from the National Wetlands Inventory in Geodatabase format from [here](https://www.fws.gov/program/national-wetlands-inventory/download-state-wetlands-data). The script uses the [leafmap](https://leafmap.org) Python package.

First, create a conda environment with the required packages:

```bash
conda create -n gdal python=3.11
conda activate gdal
conda install -c conda-forge mamba
mamba install -c conda-forge libgdal-arrow-parquet gdal leafmap
```

If you are using Google Colab, you can install the packages as follows:

```{code-cell} ipython3
# %pip install leafmap lonboard==0.3.0
```

Then, run the script below:

```{code-cell} ipython3
import leafmap
import pandas as pd

url = 'https://open.gishub.org/data/us/us_states.csv'
df = pd.read_csv(url)
ids = df['id'].tolist()
ids.sort()
urls = [f"https://documentst.ecosphere.fws.gov/wetlands/data/State-Downloads/{id}_geodatabase_wetlands.zip" for id in ids]
leafmap.download_files(urls, out_dir='.', unzip=True)
```

## Data conversion

The script below was used to convert the data from the original Geodatabase format to [Parquet](https://parquet.apache.org) format. The script uses the [leafmap](https://leafmap.org) Python package.

```{code-cell} ipython3
import leafmap
import pandas as pd

url = 'https://open.gishub.org/data/us/us_states.csv'
df = pd.read_csv(url)
ids = df['id'].tolist()

for index, state in enumerate(ids):
    print(f'Processing {state} ({index+1}/{len(ids)})')
    gdb = f"{state}_geodatabase_wetlands.gdb/"
    layer_name = f'{state}_Wetlands'
    leafmap.gdb_to_vector(gdb, ".", gdal_driver="Parquet", layers=[layer_name])
```

The total file size of the Geodatabase files is 32.5 GB. The total file size of the Parquet files is 75.8 GB.

## Data access

The script below can be used to access the data using [DuckDB](https://duckdb.org). The script uses the [duckdb](https://duckdb.org) Python package.

```{code-cell} ipython3
import duckdb

con = duckdb.connect()
con.install_extension("spatial")
con.load_extension("spatial")

state = "DC"    # Change to the US State of your choice
url = f"https://data.source.coop/giswqs/nwi/wetlands/{state}_Wetlands.parquet"
con.sql(f"SELECT * EXCLUDE geometry, ST_GeomFromWKB(geometry) FROM '{url}'")
```

Alternatively, you can use the aws cli to access the data directly from the S3 bucket:

```bash
aws s3 ls s3://us-west-2.opendata.source.coop/giswqs/nwi/wetlands/
```

## Data visualization

To visualize the data, you can use the [leafmap](https://leafmap.org) Python package with the [lonboard](https://github.com/developmentseed/lonboard) backend. The script below shows how to visualize the data.

```{code-cell} ipython3
import leafmap

state = "DC"   # Change to the US State of your choice
url = f"https://data.source.coop/giswqs/nwi/wetlands/{state}_Wetlands.parquet"
gdf = leafmap.read_parquet(url, return_type='gdf', src_crs='EPSG:5070', dst_crs='EPSG:4326')
leafmap.view_vector(gdf, get_fill_color=[0, 0, 255, 128])
```

![vector](https://i.imgur.com/HRtpiVd.png)

Alternatively, you can specify a color map to visualize the data.

```{code-cell} ipython3
color_map =  {
        "Freshwater Forested/Shrub Wetland": (0, 136, 55),
        "Freshwater Emergent Wetland": (127, 195, 28),
        "Freshwater Pond": (104, 140, 192),
        "Estuarine and Marine Wetland": (102, 194, 165),
        "Riverine": (1, 144, 191),
        "Lake": (19, 0, 124),
        "Estuarine and Marine Deepwater": (0, 124, 136),
        "Other": (178, 134, 86),
    }
leafmap.view_vector(gdf, color_column='WETLAND_TYPE', color_map=color_map, opacity=0.5)
```

![vector-color](https://i.imgur.com/Ejh8hK6.png)

Display a legend for the data.

```{code-cell} ipython3
leafmap.Legend(title="Wetland Type", legend_dict=color_map)
```

![legend](https://i.imgur.com/fxzHHFN.png)
