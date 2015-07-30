# Utah Terrain

Visualizations of Utah geographic terrain using ThreeJS. Based heavily on the work of [thematic mapping blog](http://blog.thematicmapping.org)

--------

## Demo

[View the Demo](http://cmalven.github.io/utah-terrain)

## Setup

This project uses terrain data from the [Utah AGRC (Automated Geographic Reference Center)](http://gis.utah.gov/data/elevation-terrain-data/)

### Download Terrain Data

The first thing you'll need to do is download the state-wide DEM (digital elevation model) files.

For these models we're using the [10 meter DEMs](http://gis.utah.gov/data/elevation-terrain-data/10-30-90-meter-elevation-models-usgs-dems/) which you can download the most easily from their FTP site. Once you've downloaded all of these files you'll need to expand them ([The Unarchiver](https://code.google.com/p/theunarchiver/) on Mac works great for this) and move them all into the same directory.

**After decompression, move all `.dem` files to a `utah_dem` folder on your desktop. This is where future steps will expect them to be.**

### Install GDAL

[GDAL (Geospatial Data Abstraction Libary)] is the swiss army knife that will allow us to do all of the conversion and manipulation we need to with our terrain data.

[Download GDAL Complete](http://www.kyngchaos.com/software/frameworks) and install it.

### Convert DEMs to virtual dataset

In order to make the raw terrain data easier to work with we need to convert it to a virtual dataset (`.vrt`) file that we can then convert to a GeoTIFF file.

```bash
cd /src
gdalbuildvrt -input_file_list utah_input_file_list.txt utah.vrt
```

### Convert virtual dataset to GeoTIFF

All of the remaining steps will use a GeoTIFF file as their source, which we'll create now. The `10000` value here sets the width of the output file, otherwise the file would be much, much bigger than we need.

```bash
gdalwarp -ts 10000 0 utah.vrt utah.tif
```

Additionally, you might want to create an even move scaled down version to use in generating textures later on:

```bash
gdalwarp -ts 5000 0 utah.vrt utah-small.tif
```

### Examine the GeoTIFF output

```bash
gdalinfo -mm utah.tif
```

This will tell us some interesting details about the terrain output that we'll need to work with in the next steps. The most relevant details are the `Size` and the `Computed Min/Max` values.

### Create elevation data for importing into ThreeJS

```bash
# Create a .bin of terrain data
gdal_translate -scale 0 5276 0 65535 -ot UInt16 -outsize 400 507 -of ENVI utah.tif ../assets/utah.bin

# Create a smaller version for occulus demo
gdal_translate -scale 0 5276 0 65535 -ot UInt16 -outsize 280 355 -of ENVI utah.tif ../assets/utah-lite.bin
```

`5276` is a value that is greater than the `Computed Max` value from `gdalinfo` and will be the highest elevation point in the data. `-outsize 400 507` is the output size in pixels, and should be the same ratio as the `Size` we saw in `gdalinfo` but scaled down to make the visualization less GPU intensive.

### Create the color relief and hillshade images

The following commands create color and shading images that we'll map onto our visualization to give it more depth. You can get different coloring by adjusting the values in `src/color-relief.txt`

```bash
# Create the color relief
gdaldem color-relief utah-small.tif color-relief.txt utah-relief.tif

# Create the hillshade
gdaldem hillshade -combined utah-small.tif utah-hillshade.tif
```

After you've created these two files, you'll need to combine them in Photoshop (using Multiply blending for the top layer) and export them as `utah-texture.jpg` to `/assets`

## Development

Run the following from the root of the project:

```bash
# Install BrowserSync
npm install -g browser-sync

# Start a Browsersync server
browser-sync start --server --files "**/*.html, **/*.jpg, **/*.bin"
```

Then navigate to [/texture](http://localhost:3000/texture), [/wireframe](http://localhost:3000/wireframe), or [/oculus](http://localhost:3000/oculus)