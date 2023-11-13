# National Wetlands Inventory

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

Then, run the script below:

```python
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

```python
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

```python
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

To visualize the data, you can use the [leafmap](https://leafmap.org) Python package with the [lonboard](https://github.com/developmentseed/lonboard) backend. The script below shows how to install the lonboard backend.

```bash
pip install lonboard
```

After installing the lonboard backend, you can use the script below to visualize the data.

```python
import duckdb
import leafmap.deckgl as leafmap

con = duckdb.connect()
con.install_extension("spatial")
con.load_extension("spatial")

state = "DC"   # Change to the US State of your choice
url = f"https://data.source.coop/giswqs/nwi/wetlands/{state}_Wetlands.parquet"
df = con.sql(f"SELECT * EXCLUDE geometry, ST_AsText(ST_GeomFromWKB(geometry)) AS geometry FROM '{url}'").df()
gdf = leafmap.df_to_gdf(df, src_crs="EPSG:5070", dst_crs="EPSG:4326")

m = leafmap.Map()
m.add_gdf(gdf)
m
```

![](https://i.imgur.com/nDYBWfX.png)
