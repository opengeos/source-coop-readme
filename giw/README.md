---
jupytext:
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.15.2
---

# Non-Floodplain Wetlands

[![image](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/opengeos/source-coop-readme/blob/main/giw/README.ipynb)

## Description

This dataset represents the extent and approximate location of [Geographically Isolated Wetlands](https://catalog.data.gov/dataset/geographically-isolated-wetlands-non-floodplain-wetlands-of-the-conterminous-united-states1) (GIWs), also known as non-floodplain wetlands (NFWs), in the conterminous United States. National Wetlands Inventory (NWI) lacustrine systems and palustrine wetlands were determined to be “isolated” based on their geographic location (i.e., unconnected, based on a distance measure, to specific classes of NHD aquatic systems). GIWs were here considered geographically isolated when they were outside of 10 meters from select NHD lines and polygons or were not adjacent to NWI Riverine or Estuarine wetlands and (where applicable) outside of 10 meters from a coastline (e.g., oceans or Great Lakes).

## Reference

- Lane, C. R., & D'Amico, E. (2016). Identification of putative geographically isolated wetlands of the conterminous United States. JAWRA Journal of the American Water Resources Association, 52(3), 705-722. https://doi.org/10.1111/1752-1688.12421

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

## Data download

Click on this [link](https://gaftp.epa.gov/EPADataCommons/ORD/CONUS_NFWs/Geographically_Isolated_Wetlands_of_ConterminousUnitedStates.gdb.zip) to download the data to your computer and unzip it.

## Data conversion

The script below was used to convert the data from the original Geodatabase format to [Parquet](https://parquet.apache.org) format. The script uses the [leafmap](https://leafmap.org) Python package.

```{code-cell} ipython3
import leafmap
```

```{code-cell} ipython3
gdb = 'Geographically_Isolated_Wetlands_of_ConterminousUnitedStates.gdb'
# leafmap.gdb_to_vector(gdb, ".", gdal_driver="Parquet")
```

The total file size of the Geodatabase files is 4.4 GB. The total file size of the Parquet files is 46.4 GB.

## Data access

The script below can be used to access the data using [DuckDB](https://duckdb.org). The script uses the [duckdb](https://duckdb.org) Python package.

```{code-cell} ipython3
import duckdb

con = duckdb.connect()
con.install_extension("spatial")
con.load_extension("spatial")

state = "IA"    # Change to the US State of your choice
url = f"https://data.source.coop/giswqs/giw/wetlands/{state}_IW.parquet"
# con.sql(f"SELECT * EXCLUDE geometry, ST_GeomFromWKB(geometry) as geometry FROM '{url}'")
```

Find out the total number non-floodplain wetlands in the selected state:

```{code-cell} ipython3
con.sql(f"SELECT COUNT(*) FROM '{url}'")
```

Alternatively, you can use the aws cli to access the data directly from the S3 bucket:

```bash
aws s3 ls s3://us-west-2.opendata.source.coop/giswqs/giw/wetlands/
```

## Data visualization

To visualize the data, you can use the [leafmap](https://leafmap.org) Python package with the [lonboard](https://github.com/developmentseed/lonboard) backend. The script below shows how to visualize the data.

```{code-cell} ipython3
import leafmap

state = "DC"   # Change to the US State of your choice
url = f"https://data.source.coop/giswqs/giw/wetlands/{state}_IW.parquet"
gdf = leafmap.read_parquet(url, return_type='gdf', src_crs='EPSG:5070', dst_crs='EPSG:4326')
# leafmap.view_vector(gdf, get_fill_color=[0, 0, 255, 128])
```

## Data analysis

Find out the total number non-floodplain wetlands in the conterminous United States:

```{code-cell} ipython3
con.sql(f"""
SELECT COUNT(*) AS Count FROM 's3://us-west-2.opendata.source.coop/giswqs/giw/wetlands/*.parquet'
""")
```

Find out the number of non-floodplain wetlands in each state and order them by the number of wetlands:

```{code-cell} ipython3
count_df = con.sql(f"""
SELECT inState AS State, COUNT(*) AS Count FROM 's3://us-west-2.opendata.source.coop/giswqs/giw/wetlands/*.parquet'
GROUP BY inState
ORDER BY COUNT(*) DESC;
""").df()
count_df.head()
```

Create a bar chart showing the number of non-floodplain wetlands in each state:

```{code-cell} ipython3
leafmap.pie_chart(count_df, 'State', 'Count', height=700, title='Number of Non-Floodplain Wetlands by State')
```

![](https://i.imgur.com/GgtlcWB.png)

Create a bar chart showing the number of non-floodplain wetlands in each state:

```{code-cell} ipython3
leafmap.bar_chart(count_df, 'State', 'Count', title='Number of Non-Floodplain Wetlands by State')
```

![](https://i.imgur.com/v4zz8zV.png)

Calculate the total area of non-floodplain wetlands in each state and order them by the area of wetlands:

```{code-cell} ipython3
sum_df = con.sql(f"""
SELECT inState AS State, Sum(hectares) AS Hectares FROM 's3://us-west-2.opendata.source.coop/giswqs/giw/wetlands/*.parquet'
GROUP BY inState
ORDER BY Sum(hectares) DESC;
""").df()
sum_df.head()
```

Create a pie chart showing the total area of non-floodplain wetlands in each state:

```{code-cell} ipython3
leafmap.pie_chart(sum_df, 'State', 'Hectares', height=700, title='Area of Non-Floodplain Wetlands by State')
```

![](https://i.imgur.com/mAsLCDE.png)

Create a pie chart showing the total area of non-floodplain wetlands in each state:

```{code-cell} ipython3
leafmap.bar_chart(sum_df, 'State', 'Hectares', title='Area of Non-Floodplain Wetlands by State')
```

![](https://i.imgur.com/7RtioFU.png)

Find out the mean area of non-floodplain wetlands in each state and order them by the mean area of wetlands:

```{code-cell} ipython3
median_df = con.sql(f"""
SELECT inState AS State, median(hectares)*10000 AS Meters FROM 's3://us-west-2.opendata.source.coop/giswqs/giw/wetlands/*.parquet'
GROUP BY inState
ORDER BY median(hectares) DESC;
""").df()
median_df.head(10)
```

Create a bar chart showing the median area of non-floodplain wetlands in each state:

```{code-cell} ipython3
leafmap.bar_chart(median_df, 'State', 'Meters', title='Median Area of Non-Floodplain Wetlands by State')
```

![](https://i.imgur.com/2iAYcm3.png)
