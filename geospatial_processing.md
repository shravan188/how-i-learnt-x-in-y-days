# Geospatial Processing using Python

### Goal : To learn geospatial concepts to find depth of a lake

## Day 1

### Duration : 2 hours


### Learnings

* To use Google Earth Engine api, we need to first create a non commercial cloud project by going to earthengine.google.com and click on Get Started (and then create a project)

* Once we create project with project name, we can authenticate and initialize the Earth engine Python api using code below 

```

import ee # ee = earth engine

# Trigger the authentication flow.
ee.Authenticate()

# Initialize the library.
ee.Initialize(project='my-project')

```

- In list of coordinates, first element of the coordinate list is longitude, second element is latitude.

* Bathymetry : Bathymetry is the study of the "beds" or "floors" of water bodies, including the ocean, rivers, streams, and lakes.

* GLOBathy dataset : dataset of 1.4+ million waterbodies 

* Geemap : A Python package for interactive geospatial analysis and visualization with Google Earth Engine

* Raster data : is any pixel or gridded data where each pixel or cell on the grid is associated with a specific geographical location. Each cell contains a value representing information, such as temperature

* Raster band : a band is represented by a single matrix of cell values, and a raster with multiple bands contains multiple spatially coincident matrices of cell values representing the same spatial area. An example of a single-band raster dataset is a digital elevation model (DEM). Each cell in a DEM contains only one value representing surface elevation

```
geometry = ee.Geometry(geojson)

# ee.Image is an object to represent an Earth Engine image
# the input argument is the Earth engine asset id (refer 11)

globathy = ee.Image("projects/sat-io/open-datasets/GLOBathy/GLOBathy_bathymetry")

globathy = globathy.rename('Depth_m').unmask(0)

## Visualize the global bathymetry data
Map = geemap.Map()
# depth of 0 meters is mapped to 0, depth of 20 meters is mapped to color value 255
visParams = {"min": 0, "max": 20, 'palette': ['#eff3ff','#c6dbef','#9ecae1','#6baed6','#3182bd','#08519c']}
# .clip clips an image to the goemetry coordinates given above
Map.addLayer(globathy.clip(geometry), visParams, 'Global Bathymetry') 
Map.addLayer(geometry) ## To show the bounding area of interest geometry
# tried with different zoom values
Map.centerObject(geometry, zoom=12)

## ee.Image.gt returns 1 if the value in image is greater than value in the image passed as argument
## Thus by the code below we get 1 for locations with depth greater than 0 and 0 for locations with depth equal to zero
## thus we mask the values where depths are 0
area_mask = globathy.gt(0)

## Calculate the area of each pixel in m2, rename the band as area_m2
# ee.Image.pixelArea generates an image in which the value of each pixel is the area of that pixel in square meters
# ee.Image.multiply multiplies the values in first image with second image
# first image is the 
area = area_mask.multiply(ee.Image.pixelArea()).rename('area_m2')

## Now calculate the volume of each pixel in m3, rename the band as volume_m3
volume = area.multiply(globathy).rename("volume_m3")


```


* Doubts : Do we need a GCP account to run earth engine? What information does globathy dataset contain? What is the difference between ee and geemap libraries? How to generate assetId names in Google Earth Engine? What is multiband image? How does scale and pixel in reduceRegion function affect the final output, and how do we determine their values? 

### Resources
1. https://www.youtube.com/watch?v=xTGcIFSAsSo
2. https://code.earthengine.google.com/
3. https://www.keene.edu/campus/maps/tool/
4. https://medium.com/@kavyajeetbora/unlocking-the-depths-a-guide-to-estimating-lake-volume-with-google-earth-engine-and-python-ef36b842fa2a
5. https://developers.google.com/earth-engine/apidocs/ee-image-clip#colab-python
6. https://github.com/samapriya/awesome-gee-community-datasets/tree/master (refer 11)
7. https://desktop.arcgis.com/en/arcmap/latest/manage-data/raster-and-images/what-is-raster-data.htm
8. https://desktop.arcgis.com/en/arcmap/latest/manage-data/raster-and-images/raster-bands.htm
9. https://developers.google.com/earth-engine/guides/image_visualization
10. https://developers.google.com/earth-engine/datasets (data catalog for image ids)
11. https://developers.google.com/earth-engine/datasets/catalog/projects_sat-io_open-datasets_GLOBathy_GLOBathy_bathymetry#bands
12. https://tutorials.geemap.org/Image/image_overview/
13. https://gee-community-catalog.org/projects/globathy/

## Day 2

### Duration : 1 hour

*  The scale at which to request inputs to a computation is determined from the output. For example, if you add an image to the Code Editor or geemap map element, the zoom level of the map determines the scale at which inputs are requested from the image pyramid.

* Spatial resolution : Spatial resolution (also referred to as ground sample distance GSD) refers to the size of one pixel on the ground. For example, a spatial resolution of 15 meters means that one pixel on the image corresponds to a square of 15 by 15 meters on the ground

* NIR band of a Landsat image has 30 meters native resolution. Similarly the spatial resolution for the Globathy dataset is 30m. This information is available if you go to link 4, then go the bands section, where you will see spatial resolution is 30m and thre is only 1 band

```

# add the volume values for the selected geometry, with scale 30m because spatial resolution is 30m
volume = volume.select("volume_m3")
totalVolume = volume.reduceRegion(
    reducer = ee.Reducer.sum(),
    geometry = geometry,
    scale=30,
)

totalVolume = ee.Number(totalVolume.get('volume_m3')).divide(1e3).round()
## Total volume in ML i.e. mega litres
totalVolume

```

* Doubts: How to get all values in a band?

### Resources
1. https://gis.stackexchange.com/questions/392924/what-does-it-mean-by-scale-in-reduceregion-function-in-google-earth-engine
2. https://developers.google.com/earth-engine/guides/scale
3. https://landscape.satsummit.io/capture/resolution-considerations.html
4. https://developers.google.com/earth-engine/datasets/catalog/projects_sat-io_open-datasets_GLOBathy_GLOBathy_bathymetry
5. https://gis.stackexchange.com/questions/322081/finding-all-the-values-within-a-band-in-google-earth-engine
6. https://developers.google.com/earth-engine/guides/reducers_grouping
