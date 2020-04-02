# eDNA-metadata

A Jupyter notebook tool for reproducibly gathering sample site metadata through the Google Earth Engine API.

Input: a CSV file with the coordinates of your environmental DNA sample locations and a list of all the datasets you want to sample

Output: a CSV file with the value calculated from each dataset for each sample site

## Usage
You will need a table of sample site locations in CSV format with columns `id`, `latitude`, and `longitude` (see `sample_input.csv`).

Decide which datasets you want to use. You can browse them in the [Google Earth Engine data catalog](https://developers.google.com/earth-engine/datasets). To access the dataset you will need its code snippet, which is displayed under 'Earth Engine Snippet' on the dataset's page.

### Getting started
You should run the notebook in a conda environment as is recommended for both [Jupyter Notebook](https://jupyter.readthedocs.io/en/latest/install.html#installing-jupyter-using-anaconda-and-conda) and [Google Earth Engine](https://developers.google.com/earth-engine/python_install#environments).

Install conda if you don't have it already, following the instructions [here](https://developers.google.com/earth-engine/python_install-conda#install_conda).

Make sure you are in the `eDNA-metadata/` directory. Create a new conda environment from the included environment.yml file, which will install all the necessary packages:

`conda env create -f environment.yml`

You should now have a new conda environment called `ee`. Activate it:

`conda activate ee`

Optionally, install jupyter notebook extensions, which adds some nice features like collapsing sections:

`conda install -c conda-forge jupyter_contrib_nbextensions`

You will need to do a one-time authentication which links Earth Engine to your Google account. Run:

`earthengine authenticate`

and follow the resulting instructions.

Run Jupyter notebook within the environment:

`jupyter notebook`

Make sure that the environment is activated before running `jupyter notebook`. If it's not, you could end up running the version of Jupyter notebook installed in your base environment, which won't have access to the packages you just installed.

When the Jupyter notebook tab opens, open `metadata.ipynb`, and go through the sections of the notebook to set up your datasets.

### Raster datasets (Earth Engine `Image` datasets)
To load an `ee.Image` dataset, use the `RasterDataset` class in `metadata.ipynb`.
For example, suppose we are interested in getting data of the soil pH at surface level from the [OpenLandMap Soil pH in H20 dataset](https://developers.google.com/earth-engine/datasets/catalog/OpenLandMap_SOL_SOL_PH-H2O_USDA-4C1A2A_M_v02):

Instantiate a `RasterDataset` for it, using the desired band (`b0` is the pH at 0 cm depth; other bands are for other depths), and giving it a descriptive name (optional):

```
RasterDataset(
    snippet=ee.Image("OpenLandMap/SOL/SOL_PH-H2O_USDA-4C1A2A_M/v02"),
    band='b0',
    name='soil bulk density'
)
```
Here is a false-color view of the dataset:

![soil ph map](https://github.com/emlys/eDNA-metadata/blob/master/images/soil_ph.png)

Note that if you zoom in, pixelation becomes visible, and the data does not always cover all the way to the coastline:

![soil ph zoom](https://github.com/emlys/eDNA-metadata/blob/master/images/soil_ph_zoom.png)

This is inherent to the raster format. The resolution of this dataset is 250 meters. You should be aware of the resolution you are working with and consider that in choosing a sample area radius and in reporting results. It's possible to reproject a raster dataset to a lower or higher resolution, but it's not clear if this is always the best thing to do. See the Earth Engine pages on [scale](https://developers.google.com/earth-engine/scale) and [projections](https://developers.google.com/earth-engine/projections) for more information.

### Raster dataset series (Earth Engine `ImageCollection` datasets)
Some datasets in Earth Engine consist of a collection of many images. Usually these are overlapping images from different points in time, such as from satellite data.

The individual `Image`s in a collection often do not fully cover the area of interest. So, you may need to experiment with different date ranges in order to get adequate coverage. For example, suppose we are interested in the [Landsat 8 NDVI dataset](https://developers.google.com/earth-engine/datasets/catalog/LANDSAT_LC08_C01_T1_8DAY_NDVI). Here is the dataset coverage over the continental US over one day, one month, and one year:

![date_range=('2019-01-01', '2019-01-02')](https://github.com/emlys/eDNA-metadata/blob/master/images/ndvi_day.png)
![date_range=('2019-01-01', '2019-02-01')](https://github.com/emlys/eDNA-metadata/blob/master/images/ndvi_month.png)
![date_range=('2019-01-01', '2019-12-31')](https://github.com/emlys/eDNA-metadata/blob/master/images/ndvi_year.png)

Let's say the one-month coverage is sufficient. Instantiate a `RasterDataset` using that date range:

```
RasterDataset(
    snippet=ee.ImageCollection("LANDSAT/LC08/C01/T1_8DAY_NDVI"),
    band='NDVI',
    date_range=('2019-01-01', '2019-02-01')
)
```

If the collection has more than one image in your chosen date range, they will be merged into one image using the `ee.ImageCollection.mosaic()` algorithm, and will be treated as one dataset. If you want to compare data from the same collection at different points in time, instantiate a `RasterDataset` for each date range. If you don't care so much about the time the imagery was taken, you can use a broad date range such as one year, which should provide good coverage.

You can also apply Earth Engine API functions to the dataset before passing it to the constructor. For example:

```
RasterDataset(
    snippet=ee.ImageCollection("LANDSAT/LC08/C01/T1_8DAY_NDVI").median(),
    band='NDVI',
    date_range=('2019-01-01', '2019-02-01')
)
```
Here we use the built-in `median` reducer function over the dataset, which returns an `Image` where each pixel's value is the median value across all images in the `ImageCollection` that cover that point. See the notebook for more examples.

### Vector datasets (Earth Engine `FeatureCollection` datasets)

Less commonly, data is available in vector format as an `ee.FeatureCollection` object. For example, the [EPA Level III Ecoregions dataset](https://developers.google.com/earth-engine/datasets/catalog/EPA_Ecoregions_2013_L3) is in this format. To use it, instantiate a `VectorDataset`:

```
VectorDataset(
        snippet=ee.FeatureCollection('EPA/Ecoregions/2013/L3'),
        property='us_l3name'
    )
```
`FeatureCollection`s have properties rather than bands.
