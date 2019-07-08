# RivWidthCloud

RivWidthCloud, or RWC, is an open source software written to automate extracting river centerline and width from remotely sensed images. It was developed for the Google Earth Engine (GEE) platform and developed for both its JavaScript and Python APIs. A complete description of the method can be found in our publication [RivWidthCloud: An Automated Google Earth Engine algorithm for river width extraction from remotely sensed imagery](https://ieeexplore.ieee.org/document/8752013).

## Usage guide

The easiest and quickest way to use RWC is to run it from Google Earth Engine JavaScript code editor (https://developers.google.com/earth-engine/playground), where the functions in the RWC can be directly loaded and need no setup. The README file in the RivWidthCloud_JavaScript folder contains example script to run RWC for one Landsat image. If you need to run RWC over many Landsat images, then running the Python version might be more efficient.

```JavaScript
// Goal: calculate river centerlines and widths for one Landsat SR image (LC08_L1TP_022034_20130422_20170310_01_T1)

// load in the functions
var fns = require('users/eeProject/RivWidthCloudPaper:rwc_landsat.js');

// assign the image id of the image from which the widths and centerline will be extracted
var imageId = "LC08_L1TP_022034_20130422_20170310_01_T1";

// setting the parameters for the rivwidth (rw) function
var aoi = ee.Geometry.Polygon(
        [[[-88.47207053763748, 37.46382559855354],
          [-88.47207053763748, 37.375480838211885],
          [-88.2592104302156, 37.375480838211885],
          [-88.2592104302156, 37.46382559855354]]], null, false);
var rw = fns.rwGenSR('Jones2019', 4000, 333, 500, aoi);

// apply the rw function to an image
var widths = rw(imageId);

// export the result as a csv file into Google drive
Export.table.toDrive({
  collection: widths.map(function(f) {return(f.setGeometry(null))}),
  description: imageId,
  folder: "",
  fileNamePrefix: imageId,
  fileFormat: "CSV"});
```

```bash
python rwc_landsat_one_image -h
```

## Files

The core algorithms responsible for calculating river centerlines and widths are identical in the JavaScript and the Python version. However, there is minor differences in how users might call these functions. Below is a description of the files that were common to both version. For files unique to different version please refer to the README.md file in its corresponding folder.

List of files that's common to both the JavaScript and the Python version:
* __functions_Landsat578__: contains files to process Landsat collection 1 tier 1 SR images
* __functions_Landsat578/functions_landsat__: process Landsat image to (1) add classified water mask and (2) add bands of quality flags (cloud, cloud shadow, snow/ice, hill shadow)
* __functions_Landsat578/functions_waterClassification_Zou2018__: contains water classification function based on [Zou et al., 2018](https://doi.org/10.1073/pnas.1719275115).
* __functions_Landsat578/functions_waterClassification_Jones2019__: contains water classification function based on Dynamic Surface Water Extent (DSWE) based on [Jones, 2019](https://doi.org/10.3390/rs11040374).
* __rwc_landsat__: contains a wrapper function that sets the default parameter values for the RivWidthCloud software and outputs the main RivWidthCloud function.

* __functions_centerline_width__: contains image processing functions that calculate 1px-wide centerline, crosssectional direction and widths sequentially.
* __functions_river__: contains functions that calculate channel and river mask based on a given water mask.

## Outputs

Column name|Description|Unit
-------|---------|---------
latitude|Latitude of the centerline point|Decimal degree
longitude|Longitude of the centerline point|Decimal degree
width|Wetted river width measured at the centerline point|Meter
orthogonalDirection|Angle of the cross-sectional direction at the centerline point|Radian
flag_elevation|Mean elevation across the river surface (unit: meter) based on [MERIT DEM](http://hydro.iis.u-tokyo.ac.jp/~yamadai/MERIT_DEM/)|Meter
image_id|Image ID of the input Landsat image|NA
crs|the projection of the input image|NA
flag_hillshadow|indicate potential topographic shadow nearby that could affect the width accuracy|NA
flag_snowIce|indicate potential snow/ice nearby that could affect the width accuracy|NA
flag_cloud|indicate potential cloud nearby that could affect the width accuracy|NA
flag_cldShadow|indicate potential cloud shadow nearby that could affect the width accuracy|NA
endsInWater|indicate inaccurate width due to the insufficient length of the cross-sectional segment that was used to measure the river width|NA
endsOverEdge|indicate width too close to the edge of the image that the width can be inaccurate|NA

_flag_snowIce, flag_cloud, flag_cldShadow, endsInWater, endsOverEdge, flag_hillshadow: all with values ranging from 0 to 1 with non-zero denoting that the corresponding conditions exist and the calculated width could be affected._

## Cite

Yang, X., T.M. Pavelsky, G.H. Allen, and G. Donchyts (2019), RivWidthCloud: An Automated Google Earth Engine algorithm for river width extraction from remotely sensed imagery, IEEE Geoscience and Remote Sensing Letters. DOI: 10.1109/LGRS.2019.2920225

An early access of this article can be found at: https://ieeexplore.ieee.org/document/8752013

## Contact

We welcome any feedback or suggestions for improvement. If you have any questions about the algorithm, please don't hesitate to contact yangxiao@live.unc.edu.
