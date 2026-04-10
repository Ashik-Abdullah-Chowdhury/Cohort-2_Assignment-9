//SAR SENTINEL - 1 DATA FILTERING 
 //import study area shapefile
var feni=table.filter(ee.Filter.eq("ADM2_EN","Feni")).geometry()

Map.centerObject(feni, 10);
// Map.addLayer(feni);
// Import Sentinel-1 SAR GRD
var image = ee.ImageCollection('COPERNICUS/S1_GRD')
.filter(ee.Filter.listContains('transmitterReceiverPolarisation', 'VV')) 
// Filter to get images with VV polarization “VV” stands for vertical transmit, vertical recieved.
.filter(ee.Filter.listContains('transmitterReceiverPolarisation', 'VH'))
// Filter to get images with VH polarization “VH” stands for vertical transmit, horizontal recieved.
.filter(ee.Filter.eq('instrumentMode', 'IW'))  // Filter to get images collected in interferometric wide swath mode.
.filter(ee.Filter.eq('orbitProperties_pass', 'DESCENDING')) // Also filter based on the orbit: descending or ascending mode
//.select('VV', 'VH', 'angle')
.filterMetadata('resolution_meters', 'equals' , 10)
.filterBounds(feni) 
.filterDate('2024-08-01', '2024-08-31')
// .first();

// image = image.clip(feni);
// image = image.select('VV', 'VH');
//Print Collection and add it to the map
print(image);
print(image.size());
// Map.addLayer(image, {min: -25, max: 0}, 'Search Sentine-1', true);
//mosaic Images
var mosaicS1 = image.mosaic();
print(mosaicS1);
// Map.addLayer(mosaicS1, {min: -25, max: 0}, 'Mosaic Sentinel 1', true);
//Select Bands from mosaic Collection and and calculate their difference for display
//(it will revert the colors)
var VV  = (mosaicS1.select('VV')).rename('VV');
var VH  = (mosaicS1.select('VH')).rename('VH');
var diff = ((VH).divide(VV)).rename('diffHV'); //calculate difference
var S1 = VH.addBands(VV).addBands(diff); //add bands
var S1_Viz = {min: -25, max: 5}; //visual parameters
//add it to the map by clipping it
// Map.addLayer(S1.clip(feni), S1_Viz, 'Clipped S1 feni', true);
//*******************************************************************************************************
//CALCULATING FLOOD
//Create a Cloud free S2 image (Cloud Masking)
/**
* Function to mask clouds using the Sentinel-2 QA band
* @param {ee.Image} image Sentinel-2 image
* @return {ee.Image} cloud masked Sentinel-2 image
*/
function maskS2clouds(image) {
  var qa = image.select('QA60');
  var cloudBitMask = 1 << 10; // Bits 10 and 11 are dense clouds and cirrus, respectively.
  var cirrusBitMask = 1 << 11; // Bits 10 and 11 are dense clouds and cirrus, respectively.
  var mask = qa.bitwiseAnd(cloudBitMask).eq(0) // Both flags should be set to zero, indicating clear conditions.
              .and(qa.bitwiseAnd(cirrusBitMask).eq(0));
  return image.updateMask(mask).divide(10000);
}
//Import Sentinel 2 Harmonized Collection
var dataset = ee.ImageCollection('COPERNICUS/S2_SR_HARMONIZED')
                  .filterDate('2024-01-01', '2024-07-31') /*check this*/
                  // Pre-filter to get less cloudy granules.
                  .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE',10))
                  .map(maskS2clouds); //apply cloud mask function
//Visual Parameters
var visualization = {
  min: 506,
  max: 3223,
  bands: ['B8', 'B4','B3'],
};
// Map.addLayer(dataset.median().clip(feni), visualization, 'RGB');
//Add it to the map by creating composite (mean)
// // //Calculate NDWI
var s2 = dataset.median().clip(feni);
var ndwi = s2.normalizedDifference(['B3', 'B8']);
// var ndsiViz = {min: 0, max: 1, palette: ['44c9f1', '1637f1']};
// Map.addLayer(ndsi.clip(aoi), ndsiViz, 'NDSI', false);
var ndwiViz = {min: 0.5, max: 1, palette: ['1637f1', '44c9f1']};
// Add SRTM
var srtm = ee.Image("CGIAR/SRTM90_V4").toInt();
var ndwiMasked = ndwi.updateMask(ndwi.gte(-0.1)).updateMask(srtm.lte(500));
// Map.addLayer(ndwiMasked.clip(feni), ndwiViz, 'Permanent Water');
// //Calculate Flood Water
var post_VH = S1.select('VH');
var flood = post_VH.lte(-21).and(srtm.lt(1700)); /*check this*/
var flood_mask = flood.updateMask(flood.eq(1)).updateMask(ndwi.lt(0));
var flood_viz = {palette:['Red']};
Map.addLayer(flood_mask.clip(feni),flood_viz,'Flood_Inundation');
// // //**************************************************************************************

// ==========================================
// 2. Process NASA VNP46A2 NTL Data
// ==========================================
var ntlCol = ee.ImageCollection("NASA/VIIRS/002/VNP46A2")
  .select('Gap_Filled_DNB_BRDF_Corrected_NTL');

// Helper function to calculate mean NTL for a specific time window
function getMeanNTL(start, end) {
  return ntlCol.filterDate(start, end).mean().updateMask(flood_mask)
}

// Define the three temporal windows for the flood event
var pre  = getMeanNTL('2024-01-01', '2024-07-31'); // Baseline before flood
var post = getMeanNTL('2024-08-18', '2024-08-31'); // Immediate aftermath
var rec  = getMeanNTL('2025-01-01', '2025-06-30'); // Recovery period

// Calculate NTL Recovery Index: (Recovery - Post-flood) / Pre-flood
// An offset of 0.1 is added to the denominator to prevent division by zero errors
var ntlRecovery = rec.subtract(post)
  .divide(pre.add(0.1)) 
  .rename('NTL_Recovery');


// ==========================================
// 3. Process Independent Variables (X)
// ==========================================

// Variable 1: Population Density (WorldPop)
var pop = ee.ImageCollection("WorldPop/GP/100m/pop").filterDate('2020').mean();
var popDensity = pop.rename('PopDensity').updateMask(flood_mask)

// Variable 2: Wealth Index (Meta RWI)

var rwi = ee.FeatureCollection("projects/sat-io/open-datasets/facebook/relative_wealth_index")
 .filterBounds(feni);


// // ==========================================
// // 4. Data Sampling & Preparation
// // ==========================================

// Create a multi-band image (stack) of the raster variables
// Note: Wealth is excluded here because it will be pulled from the RWI points directly
var stack = ntlRecovery
  .addBands(popDensity)

// Sample the raster stack at the exact locations of the RWI wealth points
var regressionData = stack.sampleRegions({
  collection: rwi,       // Use the RWI point locations as anchors
  properties: ['rwi'],   // Extract the raw wealth score directly from the point data
  scale: 100,            // Sample at 100m resolution (matches WorldPop)
  geometries: true       // Keep spatial coordinates for potential spatial analysis later
}).map(function(f) {
  // Add a 'constant' column filled with 1s. This is required by the OLS regression
  // algorithm to calculate the Y-intercept (β0).
  return f.set('constant', 1);
}).filter(ee.Filter.notNull(['NTL_Recovery', 'PopDensity', 'rwi'])); 
// Drop any row that contains missing data to ensure the regression runs smoothly

// // // ==========================================
// // // 5. Ordinary Least Squares (OLS) Regression
// // // ==========================================

var result = regressionData.reduceColumns({
  reducer: ee.Reducer.linearRegression({numX: 3, numY: 1}),
  selectors: ['constant', 'PopDensity', 'rwi', 'NTL_Recovery']
});

print(result);
// // ==========================================
// // 6. Output & Visualization
// // ==========================================

// Extract the calculated coefficients matrix and transpose it for easier reading
var coefficients = ee.Array(result.get('coefficients')).transpose();

print('--- Regression Results ---');
print('Intercept (β0):', coefficients.get([0,0]));
print('PopDensity Coeff (β1):', coefficients.get([0,1]));
print('Wealth (RWI) Coeff (β2):', coefficients.get([0,2])); 


// // Add raster layers to the map for visual inspection
Map.addLayer(ntlRecovery, {min: -0.5, max: 0.5, palette: ['red', 'white', 'green']}, 'NTL Recovery Index');
Map.addLayer(popDensity, {min: 0, max: 50, palette: ['white', 'purple']}, 'Population Density');
// // (Wealth image is no longer visualized here as it remains a point collection)



var scatter1 = ui.Chart.feature.byFeature({
  features: regressionData,
  xProperty: 'PopDensity',
  yProperties: ['NTL_Recovery']
})
.setChartType('ScatterChart')
.setOptions({
  title: 'Population Density vs ΔNTL',
  hAxis: {title: 'Population Density'},
  vAxis: {title: 'Δ NTL'},
  trendlines: {0: {showR2: true}},
  pointSize: 5
});

print(scatter1);

var scatter2 = ui.Chart.feature.byFeature({
  features: regressionData,
  xProperty: 'rwi',
  yProperties: ['NTL_Recovery']
})
.setChartType('ScatterChart')
.setOptions({
  title: 'Wealth (RWI) vs ΔNTL',
  hAxis: {title: 'Wealth Index (RWI)'},
  vAxis: {title: 'Δ NTL'},
  trendlines: {0: {showR2: true}},
  pointSize: 5
});

print(scatter2);