// Load Sentinel-1 images to map Ancient Cairo flooding, Egypt, October 2019-2019.

// Default location
var pt = ee.Geometry.Polygon(
 [[[31.19076202188125,29.89914686683647],
 [31.420788266998436,29.89914686683647],
 [31.420788266998436,30.11617803528595],
  [31.19076202188125,30.11617803528595],
 [31.19076202188125,29.89914686683647]]]);
// Grand Morin near Coulommiers

// Load Sentinel-1 C-band SAR Ground Range collection (log scaling, VV co-polar)
var collection = ee.ImageCollection('COPERNICUS/S1_GRD')
.filterBounds(geometry)
.filter(ee.Filter.listContains('transmitterReceiverPolarisation', 'VV'))
.select('VV');

// Filter by date
var before = collection.filterDate('2019-10-21', '2019-10-25').mosaic();
var after = collection.filterDate('2019-10-26', '2019-10-30').mosaic();


// Threshold smoothed radar intensities to identify "flooded" areas.
var SMOOTHING_RADIUS = 100; 
var DIFF_UPPER_THRESHOLD = -3;
var diff_smoothed = after.focal_median(SMOOTHING_RADIUS, 'circle', 'meters')
.subtract(before.focal_median(SMOOTHING_RADIUS, 'circle', 'meters'));
var diff_thresholded = diff_smoothed.lt(DIFF_UPPER_THRESHOLD);

// Display map

Map.centerObject(pt, 13);
Map.addLayer(before, {min:-30,max:0}, 'Before flood');
Map.addLayer(after, {min:-30,max:0}, 'After flood');
Map.addLayer(after.subtract(before), {min:-10,max:10}, 'After - before', 0);
Map.addLayer(diff_smoothed, {min:-10,max:10}, 'diff smoothed', 0);
Map.addLayer(diff_thresholded.updateMask(diff_thresholded), {palette:"0000FF"},'flooded areas - blue',1);
var hydrosheds = ee.Image('WWF/HydroSHEDS/03VFDEM');
var terrain = ee.Algorithms.Terrain(hydrosheds);
var slope = terrain.select('slope');
before = before.mask(slope.lt(5));
after = after.mask(slope.lt(5));
var before = collection.filterDate('2019-10-21', '2019-10-25')
.filter(ee.Filter.eq('orbitProperties_pass', 'DESCENDING'))
.mosaic();

Export.image.toDrive({
 image:diff_thresholded,
 description: 'Cairodiff_thresholded20191023',
 scale: 10,
 region: geometry,
 maxPixels: 3E10

});
