---
jupytext:
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.16.2
kernelspec:
  display_name: Python 3
  language: python
  name: python3
---

# Non-Floodplain Wetlands

[![image](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/opengeos/source-coop-readme/blob/main/giw/README.ipynb)

## Introduction

The datasets represent the extent and approximate location of [Geographically Isolated Wetlands](https://catalog.data.gov/dataset/geographically-isolated-wetlands-non-floodplain-wetlands-of-the-conterminous-united-states1) (GIWs), also known as non-floodplain wetlands (NFWs), in the conterminous United States. National Wetlands Inventory (NWI) lacustrine systems and palustrine wetlands were determined to be “isolated” based on their geographic location (i.e., unconnected, based on a distance measure, to specific classes of NHD aquatic systems). GIWs were here considered geographically isolated when they were outside of 10 meters from select NHD lines and polygons or were not adjacent to NWI Riverine or Estuarine wetlands and (where applicable) outside of 10 meters from a coastline (e.g., oceans or Great Lakes).

The datasets have two versions:

- [Version 1](https://beta.source.coop/giswqs/giw/wetlands) (46.37 GB): The original Lane & D'Amico (2016) dataset with the original attributes. We call this dataset **NFW**.
- [Version 2](https://beta.source.coop/giswqs/giw/wetlands_v2) (90.18 GB): The original Lane & D'Amico (2016) dataset with additional attributes from the 10-m resolution depression dataset. We call this dataset **Wetland Depressions**.

## Reference

- Lane, C. R., & D'Amico, E. (2016). Identification of putative geographically isolated wetlands of the conterminous United States. _JAWRA Journal of the American Water Resources Association_, 52(3), 705-722. https://doi.org/10.1111/1752-1688.12421

## Environment setup

First, create a conda environment with the required packages:

```bash
conda create -n gdal python=3.11
conda activate gdal
conda install -c conda-forge mamba
mamba install -c conda-forge libgdal-arrow-parquet gdal leafmap lonboard
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
gdb = "Geographically_Isolated_Wetlands_of_ConterminousUnitedStates.gdb"
# leafmap.gdb_to_vector(gdb, ".", gdal_driver="Parquet")
```

The total file size of the Geodatabase files is 4.4 GB. The total file size of the Parquet files is 46.37 GB.

## Data access

The script below can be used to access the data using [DuckDB](https://duckdb.org). The script uses the [duckdb](https://duckdb.org) Python package.

```{code-cell} ipython3
import duckdb

con = duckdb.connect()
con.install_extension("spatial")
con.load_extension("spatial")

state = "IA"  # Change to the US State of your choice
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

state = "DC"  # Change to the US State of your choice
url = f"https://data.source.coop/giswqs/giw/wetlands/{state}_IW.parquet"
gdf = leafmap.read_parquet(
    url, return_type="gdf", src_crs="EPSG:5070", dst_crs="EPSG:4326"
)
leafmap.view_vector(gdf, get_fill_color=[0, 0, 255, 128])
```

## Data analysis

Find out the total number of NFWs in CONUS:

```{code-cell} ipython3
con.sql(
    f"""
SELECT COUNT(*) AS Count
FROM 's3://us-west-2.opendata.source.coop/giswqs/giw/wetlands/*.parquet'
"""
)
```

The total number of NFWs in CONUS is 8,380,620.

+++

Find out the number of NFWs in each state and order them by the number of NFWs:

```{code-cell} ipython3
giw_count_df = con.sql(
    f"""
SELECT inState AS State, COUNT(*) AS Count
FROM 's3://us-west-2.opendata.source.coop/giswqs/giw/wetlands/*.parquet'
GROUP BY inState
ORDER BY COUNT(*) DESC;
"""
).df()
giw_count_df.head()
```

Create a pie chart showing the number of NFWs in each state:

```{code-cell} ipython3
leafmap.pie_chart(
    giw_count_df,
    "State",
    "Count",
    height=700,
    title="Number of Non-Floodplain Wetlands (NFWs) by State",
)
```

![](https://i.imgur.com/SNpVzcC.png)

Create a bar chart showing the number of NFWs in each state:

```{code-cell} ipython3
leafmap.bar_chart(
    giw_count_df, "State", "Count", title="Number of Non-Floodplain Wetlands (NFWs) by State"
)
```

![](https://i.imgur.com/19gxoBE.png)

Calculate the total area of NFWs in each state and order them by the area of wetlands:

```{code-cell} ipython3
sum_df = con.sql(
    f"""
SELECT inState AS State, Sum(hectares) AS Hectares
FROM 's3://us-west-2.opendata.source.coop/giswqs/giw/wetlands/*.parquet'
GROUP BY inState
ORDER BY Sum(hectares) DESC;
"""
).df()
sum_df.head()
```

Create a pie chart showing the total area of NFWs in each state:

```{code-cell} ipython3
leafmap.pie_chart(
    sum_df,
    "State",
    "Hectares",
    height=700,
    title="Percentage Area of Non-Floodplain Wetlands (NFWs) by State",
)
```

![](https://i.imgur.com/dayPfZY.png)

Create a pie chart showing the total area of NFWs in each state:

```{code-cell} ipython3
leafmap.bar_chart(
    sum_df, "State", "Hectares", title="Total Area of Non-Floodplain Wetlands (NFWs) by State"
)
```

```{code-cell} ipython3
median_df = con.sql(
    f"""
SELECT inState AS State, median(hectares)*10000 AS Meters
FROM 's3://us-west-2.opendata.source.coop/giswqs/giw/wetlands/*.parquet'
GROUP BY inState
ORDER BY median(hectares) DESC;
"""
).df()
median_df.head(10)
```

Create a bar chart showing the median area of NFWs in each state:

```{code-cell} ipython3
leafmap.bar_chart(
    median_df,
    "State",
    "Meters",
    title="Median Area of Non-Floodplain Wetlands (NFWs) by State",
)
```

![](https://i.imgur.com/OZahOKx.png)

+++

Calculate the number of NFWs intersecting surface depressions (i.e., wetland depressions) in each state:

```{code-cell} ipython3
giw_dep_count_stat = con.sql(
    f"""
SELECT inState AS State, COUNT(DISTINCT Final_ID) AS Count
FROM 's3://us-west-2.opendata.source.coop/giswqs/giw/wetlands_v2/*.parquet'
WHERE area IS NOT NULL
GROUP BY inState
ORDER BY COUNT(*) DESC;
"""
).df()
giw_dep_count_stat.head()
```

Merge the NFW table with the wetland depressions table:

```{code-cell} ipython3
merged_df = giw_count_df.merge(giw_dep_count_stat, on="State")
merged_df.columns = ["State", "GIW_Count", "DEP_Count"]
merged_df["Percent"] = merged_df["DEP_Count"] / merged_df["GIW_Count"] * 100
merged_df.head()
```

Compare the number of NFWs and wetland depressions in each state:

```{code-cell} ipython3
leafmap.bar_chart(
    merged_df,
    "State",
    ["GIW_Count", "DEP_Count"],
    title="Number of NFWs and Wetland Depressions by State",
)
```

![](https://i.imgur.com/BsPdZ82.png)

+++

Compute the statistics of the area of NFWs and wetland depressions in each state:

```{code-cell} ipython3
giw_dep_count_stat = con.sql(
    f"""
SELECT inState AS State, COUNT(DISTINCT Final_ID) AS Count,
SUM(area) / 1e6 AS Total_Area_km2,
SUM(volume) / 1e9 AS Total_Volume_km3,
Median(area) AS Median_Area_m2,
Median(volume) AS Median_Volume_m3
FROM 's3://us-west-2.opendata.source.coop/giswqs/giw/wetlands_v2/*.parquet'
WHERE area IS NOT NULL
GROUP BY inState
ORDER BY COUNT(*) DESC;
"""
).df()
giw_dep_count_stat.head(10)
```

The median size of wetland depressions in each state:

```{code-cell} ipython3
leafmap.bar_chart(
    giw_dep_count_stat[giw_dep_count_stat['State'] != "NV"],
    "State",
    "Median_Area_m2",
    title="The Median Size of Maximum Depression Area by State",
)
```

![](https://i.imgur.com/nkD0NSM.png)

+++

The median maximum storage volume of wetland depressions in each state:

```{code-cell} ipython3
leafmap.bar_chart(
    giw_dep_count_stat,
    "State",
    "Median_Volume_m3",
    title="The Median Maximum Storage Volume of Wetlands Depressions by State",
)
```

![](https://i.imgur.com/OLohMJ9.png)
