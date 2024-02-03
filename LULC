var basinAreaOutline = ee.Image().byte().paint({ 
featureCollection: AOI, 
color: 1, 
width: 4 
}); 
Map.addLayer(basinAreaOutline, { 
palette: 'red'}, 'basin');

var filtered = s2.filter(ee.Filter.bounds(AOI))
                .filter(ee.Filter.date('2021-12-01', '2022-05-31'))
                .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE',20))
                .select('B.*')
var before = filtered.median().clip(AOI)

Export.image.toDrive({
  image: before, 
  description: 'Sentinel_2_B4-B2',
  fileNamePrefix: 'S2_B4-B2_raw',
  region: AOI,
  maxPixels: 1e10
})
Map.addLayer(before, BVisParam, 'before', false)

var after = s2.filter(ee.Filter.bounds(AOI))
                .filter(ee.Filter.date('2022-06-01', '2022-12-30'))
                .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE',20))
                .select('B.*')
                .median()
                .clip(AOI)
Map.addLayer(after, AVisParam, 'after', false)
Export.image.toDrive({
  image: after, 
  description: 'After_S2',
  fileNamePrefix: 'AS2_B4-B2_raw',
  region: AOI,
  maxPixels: 1e10
})

//Before
var training = urban.merge(bareland).merge(water).merge(vegetation)

var training = before.sampleRegions({
  collection:training , 
  properties:['Class'], 
  scale: 10})
  
  print(training)

var classifier = ee.Classifier.smileRandomForest(50).train({
  features:training ,
  classProperty:'Class',
  inputProperties: before.bandNames()})
  
var beforeClassified = before.classify(classifier)

Map.addLayer(beforeClassified, {min: 0, max: 3, palette:['blue', 'red', 'yellow', 'green']}, 'before Classified', false)
Export.image.toDrive({
  image: beforeClassified, 
  description: 'Before_Classified',
  fileNamePrefix: 'Before_Classified_raw',
  region: AOI,
  maxPixels: 1e10
})

// After
var training_after = urban_after.merge(bareland_after).merge(water_after).merge(vegetation_after)

var training_after = after.sampleRegions({
  collection:training_after , 
  properties:['Class'], 
  scale: 10})
  
  print(training_after)

var classifier_after = ee.Classifier.smileRandomForest(50).train({
  features:training_after ,
  classProperty:'Class',
  inputProperties: after.bandNames()})
  
var afterClassified = after.classify(classifier_after)

Map.addLayer(afterClassified, {min: 0, max: 3, palette:['blue', 'red', 'yellow', 'green']}, 'after Classified', false)
Export.image.toDrive({
  image: afterClassified,
  description: 'After_Classified',
  fileNamePrefix: 'After_Classified_raw',
  region: AOI,
  maxPixels: 1e10
})

// Add a random column to each feature
var withRandom = training.randomColumn('random')
// Split the points into training and validation sets
var split = 0.7;  // 70% training, 30% testing
var trainingSet = withRandom.filter(ee.Filter.lt('random', split))
var validationSet = withRandom.filter(ee.Filter.gte('random', split))

// Perform an accuracy assessment
var validated = validationSet.classify(classifier);
var testAccuracy = validated.errorMatrix('Class', 'classification').accuracy();
var confusionMatrix = validated.errorMatrix('Class', 'classification')
print('Validation error Martix: ', confusionMatrix)
print('Validation overall accuracy: ', testAccuracy)

print('Confusion Matrix', confusionMatrix);

print('Before Overall Accuracy', testAccuracy)


// Add a random column to each feature
var withRandom = training_after.randomColumn('random')
// Split the points into training and validation sets
var split = 0.7;  // 70% training, 30% testing
var trainingSet = withRandom.filter(ee.Filter.lt('random', split))
var validationSet = withRandom.filter(ee.Filter.gte('random', split))

// Perform an accuracy assessment
var validated = validationSet.classify(classifier);
var testAccuracy = validated.errorMatrix('Class', 'classification').accuracy();
var confusionMatrix = validated.errorMatrix('Class', 'classification')
print('Validation error Martix: ', confusionMatrix)
print('Validation overall accuracy: ', testAccuracy)

print('Confusion Matrix', confusionMatrix);

print('After Overall Accuracy', testAccuracy)
