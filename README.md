# Citizen science plant observations encode global trait patterns

This repository hosts additional material accompanying the manuscript *Citizen science plant observations encode global trait patterns*.

  1. The entire workflow documentation: https://sojwolf.github.io/iNaturalist_traits/
  2. GeoTiff trait maps based on iNaturalist observations and sPlotOpen observations, respectively

## Trait maps

### Versions

The maps available here were generated using iNaturalist observations downloaded on January 4, 2022 (https://doi.org/10.15468/dl.34tjre) and sPlotOpen version 52.

If you would like to update the iNaturalist maps using the most recent data, please refer to the section **Create updated trait maps** below.

1. iNaturalist

        1. 0.5 degrees

              1. ln
              2. exp(ln)

        2. 2 degrees

              1. ln
              2. exp(ln)

2. sPlotOpen

        1. 0.5 degrees

              1. ln
              2. exp(ln)

        2. 2 degrees

              1. ln
              2. exp(ln)

### Load in Python

Get file names:

```python
from os import listdir
from os.path import isfile, join

path = "iNaturalist_traits-main/iNat_maps/2_deg/ln/"
files = [f for f in listdir(path) if isfile(join(path, f))]
files.sort()
```
Load all trait maps as xarray:
```python
from import xarray as xr

def cubeFile(file):
    name = file.replace(".tif","")
    sr = xr.open_dataset(path + file,engine = "rasterio",chunks = 1024).sel(band = 1)
    sr = sr.assign_coords({"variable":name})
    return sr

da = xr.concat([cubeFile(x) for x in files],dim = "variable")
```
Select a specific band:
```
da.band_data.sel(variable = "iNat_Leaf.Area_2_ln")
```
Convert to data frame

```python
df = da_2.band_data.to_dataset().to_dataframe().reset_index()
df_spread = df.pivot(index= ['x','y'],columns='variable',values='band_data').reset_index()
```


### Load in R


## Create updated trait maps

If you would like to use the most recent vascular plant data, instead of the maps provided here, the following instructions will guide you.

### Download iNaturalist database

Open the following link and click ‘Rerun Query’:

https://doi.org/10.15468/dl.34tjre

Request the download: Click on "Download" and sign into/generate your account. For this analysis the ‘simple’ version is sufficient (roughly 10 GB). This will also generate a new DOI that you can cite. You will receive a download link via e-mail as soon as the download is ready (usually after about 15 minutes).

### Download TRY data

To download data from the TRY database, create an account at https://www.try-db.org/de.

When asked which traits you would like to download, type in the following list. This filters TRY data for our traits of interest:

```
3113, 3117, 4, 13, 14, 15, 3106, 26, 27, 47, 50, 56, 78, 138, 163, 169, 237, 282
```

Continue without species list and choose open trait data only. You will receive a download link via e-mail after about one or two days.

The iNaturalist and TRY downloads will contain files that look something like this:

iNaturalist_filename: ```0354963-210914110416597.csv```
TRY_filename: ```19287.txt```

### Link iNaturalist and TRY and generate trait maps

Download the ```make_traitmaps/``` folder and execute the containing python script to generate 0.5 and 2 degree maps.

```
python make_traitmaps.py -n iNaturalist_filename -t TRY_filename -f False
```

Run-time is around 2 hours. If you would like to apply a basic fuzzy name matching, use -f True.
Follow the more detailed workflow as documented in https://sojwolf.github.io/iNaturalist_traits/.

### Requirements

**Python packages**:

For handling data frames and (multidimensional) arrays:
  - ```pandas```
  - ```numpy```

For fuzzy matching, if you would like to use this feature:
  - ```rapidfuzz```

For system interface:
  - ```getopt```
  - ```sys```

**R packages**:

For making rasters and exporting geotiffs:
  - ```raster```
  - ```rgdal```
