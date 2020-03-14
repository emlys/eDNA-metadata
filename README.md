# eDNA-metadata

A Jupyter notebook tool for reproducibly gathering sample site metadata through the Google Earth Engine API.

Input: a CSV file with the coordinates of your environmental DNA sample locations and a list of all the datasets you want to sample

Output: a CSV file with the value calculated from each dataset for each sample site

## Usage
You will need a table of sample site locations in CSV format with columns `id`, `latitude`, and `longitude`.

Decide which datasets you want to use. You can browse them in the [Google Earth Engine data catalog](https://developers.google.com/earth-engine/datasets). To access the dataset you will need its path, which is in quotation marks in the 'Earth Engine Snippet' on the dataset's page.

### Raster datasets (Earth Engine `Image` datasets)
To load an `ee.Image` dataset, use the `RasterDataset` class in `metadata.ipynb`.
For example, suppose we are interested in getting data of the soil pH at surface level from the [OpenLandMap Soil pH in H20 dataset](https://developers.google.com/earth-engine/datasets/catalog/OpenLandMap_SOL_SOL_PH-H2O_USDA-4C1A2A_M_v02):

- The code snippet on the dataset page is `ee.Image("OpenLandMap/SOL/SOL_PH-H2O_USDA-4C1A2A_M/v02")`.
- The path to the dataset is `OpenLandMap/SOL/SOL_PH-H2O_USDA-4C1A2A_M/v02`.

Instantiate a `RasterDataset` for it, using the desired band (`b0` is the pH at 0 cm depth; other bands are for other depths), and giving it a descriptive name (optional):

```
RasterDataset(
    path='OpenLandMap/SOL/SOL_PH-H2O_USDA-4C1A2A_M/v02',
    band='b0',
    name='soil bulk density'
)
```
Here is a false-color view of the dataset:

![soil ph map](https://github.com/emlys/eDNA-metadata/blob/master/images/soil_ph.png)

Note that if you zoom in, pixelation becomes visible, and the data does not always cover all the way to the coastline:

![soil ph zoom](https://github.com/emlys/eDNA-metadata/blob/master/images/soil_ph_zoom.png)

This is inherent to the raster format. The resolution of this dataset is 250 meters.

### Raster dataset series (Earth Engine `ImageCollection` datasets)
Some datasets in Earth Engine consist of a collection of many images. Usually these are overlapping images from different points in time, such as from satellite data. To use an `ImageCollection` datase, use the `RasterDatasetSeries` class in `metadata.ipynb`. This is very similar to `RasterDataset`, but we need to specify a range of dates to look at. 

The individual `Image`s in a collection often do not fully cover the area of interest. So, you may need to experiment with different date ranges in order to get adequate coverage. For example, suppose we are interested in the [Landsat 8 NDVI dataset](https://developers.google.com/earth-engine/datasets/catalog/LANDSAT_LC08_C01_T1_8DAY_NDVI). Here is the dataset coverage over the continental US over one day, one month, and one year:

![date_range=('2019-01-01', '2019-01-02')](https://github.com/emlys/eDNA-metadata/blob/master/images/ndvi_day.png)
![date_range=('2019-01-01', '2019-02-01')](https://github.com/emlys/eDNA-metadata/blob/master/images/ndvi_month.png)
![date_range=('2019-01-01', '2019-12-31')](https://github.com/emlys/eDNA-metadata/blob/master/images/ndvi_year.png)

Let's say the one-month coverage is sufficient. Instantiate a `RasterDatasetSeries` using that date range:

```
RasterDatasetSeries(
    path='LANDSAT/LC08/C01/T1_8DAY_NDVI',
    band='NDVI',
    date_range=('2019-01-01', '2019-02-01')
)
```

If the collection has more than one image in your chosen date range, they will be merged into one image using the `ee.ImageCollection.mosaic()` algorithm, and will be treated as one dataset. If you want to compare data from the same collection at different points in time, instantiate a `RasterDatasetSeries` for each date range. If you don't care so much about the time the imagery was taken, you can use a broad date range such as one year, which should provide good coverage.


### Vector datasets (Earth Engine `FeatureCollection` datasets)
