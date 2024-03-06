//print(AOI.size()); 
//Map.addLayer(basin, {'color': 'white'}, 'basin')
Map.centerObject(AOI, 6);
Map.setOptions('Terrain');

var basinAreaOutline = ee.Image().byte().paint({ 
featureCollection: basin,
color: 1, 
width: 4 
}); 
Map.addLayer(basinAreaOutline, { 
palette: 'red'}, 'basin');

var water = gsw.select('occurrence').clip(AOI);
Map.addLayer(water, { min: 0, max: 100, palette: ['white', 'cyan', 'blue' ]}, 'Water', false);

var permanent = water.gt(80);
Map.addLayer(permanent.selfMask(), { palette: 'blue' }, 'Permanent Water', false);

var rainbow = ['blue', 'cyan', 'green', 'yellow', 'red'];

var distance = permanent.fastDistanceTransform().divide(30).clip(AOI).reproject('EPSG:4326', null, 30);
Map.addLayer(distance, { max: 0, min: 5000, palette: rainbow}, 'Distance', false);

var onlyDistance = distance.updateMask(distance.neq(0).and(srtm.mask()));
Map.addLayer(onlyDistance, { min: 0, max: 5000, palette: rainbow}, 'Distance from PW', false);

var distanceScore = onlyDistance.where(onlyDistance.gt(4000), 1)
  .where(onlyDistance.gt(3000).and(onlyDistance.lte(4000)), 2)
  .where(onlyDistance.gt(2000).and(onlyDistance.lte(3000)), 3)
  .where(onlyDistance.gt(1000).and(onlyDistance.lte(2000)), 4)
  .where(onlyDistance.lte(1000), 5);
Map.addLayer(distanceScore, { min: 1, max: 5, palette: rainbow }, 'Distance HS', false);

var elevation = srtm.clip(AOI);
Map.addLayer(elevation, { min: 0, max: 100, palette: ['green', 'yellow', 'red', 'white'] }, 'DEM', false);


var elevScore = elevation.updateMask(distance.neq(0)).where(elevation.gt(20), 1)
  .where(elevation.gt(15).and(elevation.lte(20)), 2)
  .where(elevation.gt(10).and(elevation.lte(15)), 3)
  .where(elevation.gt(5).and(elevation.lte(10)), 4)
  .where(elevation.lte(5), 5);
Map.addLayer(elevScore, { min: 1, max: 5, palette: rainbow }, 'Elevation HS', false);

var tpi = elevation.subtract(elevation.focalMean(5).reproject('EPSG:4326', null, 30)).rename('TPI');
Map.addLayer(tpi, { min: -5, max: 5, palette: ['blue', 'yellow', 'red'] }, 'TPI', false);

Export.image.toDrive({
  image: tpi, 
  description: 'tpi',
  fileNamePrefix: 'TPI',
  region: AOI,
  maxPixels: 1e10
})

var topoScore = tpi.updateMask(distance.neq(0)).where(tpi.gt(0), 1)
  .where(tpi.gt(-2).and(tpi.lte(0)), 2)
  .where(tpi.gt(-4).and(tpi.lte(-2)), 3)
  .where(tpi.gt(-6).and(tpi.lte(-4)), 4)
  .where(tpi.lte(-8), 5);
Map.addLayer(topoScore, { min: 1, max: 5, palette: rainbow }, 'Topographic HS', false);

var Alandsat8 = l8.filterBounds(AOI).filterDate('2022-06-01', '2022-10-30').median().clip(AOI).select('B.*');
Map.addLayer(Alandsat8, { min: [0.1, 0.05, 0], max: [0.4, 0.3, 0.15], bands: ['B5', 'B6', 'B3']}, 'After Landsat 8', false);
Export.image.toDrive({
  image: Alandsat8, 
  description: 'After_L8',
  fileNamePrefix: 'After_L8_raw',
  region: AOI,
  maxPixels: 100e9
});
var Blandsat8 = l8.filterBounds(AOI).filterDate('2022-03-01', '2022-05-31').median().clip(AOI).select('B.*');
Map.addLayer(Blandsat8, { min: [0.1, 0.05, 0], max: [0.4, 0.3, 0.15], bands: ['B5', 'B6', 'B3']}, 'Before Landsat 8', false);
Export.image.toDrive({
  image: Blandsat8, 
  description: 'Before_L8',
  fileNamePrefix: 'Before_L8_raw',
  region: AOI,
  maxPixels: 100e9
});
// After Band
var bandMap = {
  NIR: Alandsat8.select('B5'),
  GREEN: Alandsat8.select('B3'),
};

// After NDWI
var Andwi = Alandsat8.expression('(GREEN - NIR) / (GREEN + NIR)', bandMap).rename('A_NDWI');
Map.addLayer(Andwi, { min: -1, max: 1, palette: rainbow }, 'After NDWI', false);
Export.image.toDrive({
  image: Andwi, 
  description: 'After_NDWI',
  fileNamePrefix: 'After_NDWI_raw',
  region: AOI,
  maxPixels: 1e10
});
// After Wetness score
var A_wetScore = Andwi.updateMask(distance.neq(0)).where(Andwi.gt(0.6), 5)
  .where(Andwi.gt(0.2).and(Andwi.lte(0.6)), 4)
  .where(Andwi.gt(-0.2).and(Andwi.lte(0.2)), 3)
  .where(Andwi.gt(-0.6).and(Andwi.lte(-0.2)), 2)
  .where(Andwi.lte(-0.6), 1);
Map.addLayer(A_wetScore, { min: 1, max: 5, palette: rainbow }, 'After WHS', false);
Export.image.toDrive({
  image: A_wetScore, 
  description: 'After_WHS',
  fileNamePrefix: 'After_WHS_raw',
  region: AOI,
  maxPixels: 1e10
});
// Before Band
var bandMap = {
  NIR: Blandsat8.select('B5'),
  GREEN: Blandsat8.select('B3'),
};

// Before NDWI
var Bndwi = Blandsat8.expression('(GREEN - NIR) / (GREEN + NIR)', bandMap).rename('B_NDWI');
Map.addLayer(Bndwi, { min: -1, max: 1, palette: rainbow }, 'Before NDWI', false);
Export.image.toDrive({
  image: Bndwi, 
  description: 'Before_NDWI',
  fileNamePrefix: 'Before_NDWI_raw',
  region: AOI,
  maxPixels: 1e10
});
// Before Wetness score
var B_wetScore = Bndwi.updateMask(distance.neq(0)).where(Bndwi.gt(0.6), 5)
  .where(Bndwi.gt(0.2).and(Bndwi.lte(0.6)), 4)
  .where(Bndwi.gt(-0.2).and(Bndwi.lte(0.2)), 3)
  .where(Bndwi.gt(-0.6).and(Bndwi.lte(-0.2)), 2)
  .where(Bndwi.lte(-0.6), 1);
Map.addLayer(B_wetScore, { min: 1, max: 5, palette: rainbow }, 'Before WHS', false);
Export.image.toDrive({
  image: B_wetScore, 
  description: 'Before_WHS',
  fileNamePrefix: 'Before_WHS_raw',
  region: AOI,
  maxPixels: 1e10
});
// Flood hazard
var floodHazard = distanceScore.add(topoScore).add(A_wetScore).add(B_wetScore).add(elevScore).rename('Flood_hazard');
Map.addLayer(floodHazard, { min: 1, max: 20, palette: rainbow }, 'Flood hazard', false);

// Flood hazard scored
var floodHazardScore = floodHazard.where(floodHazard.gt(15), 5)
  .where(floodHazard.gt(10).and(floodHazard.lte(15)), 4)
  .where(floodHazard.gt(5).and(floodHazard.lte(10)), 3)
  .where(floodHazard.gt(0).and(floodHazard.lte(5)), 2)
  .where(floodHazard.lte(0), 1);
Map.addLayer(floodHazardScore, { min: 1, max: 5, palette: rainbow }, 'Flood HS', false);

// Export flood area as TIFF file 
Export.image.toDrive({
  image: floodHazard, 
  description: 'Flood_Hazard_raster',
  fileNamePrefix: 'floodHazard',
  region: AOI,
  maxPixels: 1e10
});

Export.image.toDrive({
  image: floodHazardScore, 
  description: 'Flood_Hazard_S_raster',
  fileNamePrefix: 'floodHazard_Score',
  region: AOI,
  maxPixels: 1e10
});
// A. Speckle Filter
var smoothingRadius = 50;

var difference = Andwi.focal_median(smoothingRadius, 'circle', 'meters')
  .divide(Bndwi.focal_median(smoothingRadius, 'circle', 'meters'));

var diffThreshold = 1.25;
var flooded = difference.gt(diffThreshold).rename('water').selfMask();

// B. Mask out permanent/semi-permanent water bodies
var permanentWater = ee.Image("JRC/GSW1_4/GlobalSurfaceWater")
  .select('seasonality').gte(10).clip(AOI);

flooded = flooded.where(permanentWater, 0).selfMask();

// C. Mask out areas with steep slopes
var slopeThreshold = 5;
var terrain = ee.Algorithms.Terrain(ee.Image("WWF/HydroSHEDS/03VFDEM"));
var slope = terrain.select('slope');
flooded = flooded.updateMask(slope.lt(slopeThreshold));

// D. Remove isolated pixels
var connectedPixelThreshold = 8;
var connections = flooded.connectedPixelCount();
flooded = flooded.updateMask(connections.gt(connectedPixelThreshold));

Map.addLayer(flooded, { min: 0, max: 1, palette: ['red'] }, 'Flood Extent');

Export.image.toDrive({
  image: flooded, 
  description: 'Flood_Extent_raster',
  fileNamePrefix: 'flooded',
  region: AOI,
  maxPixels: 1e10
});

//E. Calculate Flood Area
var flood_stats = flooded.multiply(ee.Image.pixelArea()).reduceRegion({
  reducer: ee.Reducer.sum(),
  geometry: AOI,
  scale: 10,
  maxPixels: 1e12,
});

var floodAreaHa = ee.Number(flood_stats.get('water')).divide(10000).round();
print('Flooded Area (Ha)', floodAreaHa);

// A. European Space Agency’s landcover dataset

var landcover = ee.ImageCollection("ESA/WorldCover/v200");
print(landcover, 'Land Cover');

var lc = landcover.mosaic().clip(AOI);

var dict = {
  "names": ["Tree cover", "Shrubland", "Grassland", "Cropland", "Built-up", "Bare / sparse vegetation", 
    "Snow and ice", "Permanent water bodies", "Herbaceous wetland", "Mangroves", "Moss and lichen"],

  "colors": ["006400", "ffbb22", "ffff4c", "f096ff", "fa0000", "b4b4b4", "f0f0f0", 
    "0064c8", "0096a0", "00cf75", "fae6a0"]
};

Map.addLayer(lc, { min: 10, max: 100, palette: dict['colors'] }, 'ESA Earth Cover');


// B. Cropland Exposed
var cropland = lc.select('Map').eq(40).selfMask();
var cropland_affected = flooded.updateMask(cropland).rename('crop');

// Calculate the area of affected Vegetation in hectares
var crop_pixelarea = cropland_affected.multiply(ee.Image.pixelArea());
var crop_stats = crop_pixelarea.reduceRegion({
  reducer: ee.Reducer.sum(),
  geometry: AOI,
  scale: 10,
  maxPixels: 1e12,
});

var floodAffectedCroplandAreaHa = ee.Number(crop_stats.get('crop')).divide(10000).round();
print('Flood Affected Cropland Area (Ha)', floodAffectedCroplandAreaHa);

// C. Built-up Exposed
var builtup = lc.select('Map').eq(50).selfMask();
var builtup_affected = flooded.updateMask(builtup).rename('builtup');

// Calculate the area of affected built-up areas in hectares
var builtup_pixelarea = builtup_affected.multiply(ee.Image.pixelArea());
var builtup_stats = builtup_pixelarea.reduceRegion({
  reducer: ee.Reducer.sum(),
  geometry: AOI,
  scale: 10,
  maxPixels: 1e12,
});

var floodAffectedBuiltupAreaHa = ee.Number(builtup_stats.get('builtup')).divide(10000).round();
print('Flood Affected Built-up Area (Ha)', floodAffectedBuiltupAreaHa);

// D. Population Exposed
var population_count = ee.Image("JRC/GHSL/P2016/POP_GPW_GLOBE_V1/2015").clip(AOI);
var population_exposed = population_count.updateMask(flooded).selfMask();

var stats = population_exposed.reduceRegion({
  reducer: ee.Reducer.sum(),
  geometry: AOI,
  scale: 250,
  maxPixels: 1e9,
});

var numberPeopleExposed = stats.getNumber('population_count').round();
print('Number of People Exposed', numberPeopleExposed);
