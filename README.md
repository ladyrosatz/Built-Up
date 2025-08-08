# Built-Up
Urban Built-Up Area Detection

## PRUEBA 1: NBUI
// ROI
var roi = geometry;

// Colección Landsat 8 SR (Level 2)
var collection = ee.ImageCollection("LANDSAT/LC08/C02/T1_L2")
  .filterBounds(roi)
  .filterDate('2024-01-01', '2024-02-29')
  .filter(ee.Filter.lt('CLOUD_COVER', 10));

var image = collection.median();

// Extraer bandas necesarias
var B2 = image.select('SR_B2'); // Green
var B3 = image.select('SR_B3'); // Red
var B4 = image.select('SR_B4'); // NIR
var B5 = image.select('SR_B5'); // SWIR1
var B6 = image.select('ST_B10'); // Thermal (Kelvin)

// Parámetro I = 1 (vegetación baja)
var I = ee.Number(1);

// NBUI según fórmula corregida
var term1 = B5.subtract(B4).divide(B5.add(B6).sqrt().multiply(10));
var term2 = B4.subtract(B3).multiply(I.add(1)).divide(B4.subtract(B3).add(1));
var term3 = B2.subtract(B5).divide(B2.add(B5));
var nbui = term1.subtract(term2.add(term3)).rename('NBUI');

// NDVI para enmascarar zonas no urbanas
var ndvi = image.normalizedDifference(['SR_B5', 'SR_B4']);
var mask = ndvi.gt(0.09); // Ajusta si es necesario

// Aplicar la máscara y recortar al ROI
var nbuiMasked = nbui.updateMask(mask).clip(roi);

// Visualizar NBUI
Map.centerObject(roi, 10);
Map.addLayer(nbuiMasked, {
  min: -0.5,
  max: 0.5,
  palette: ['blue', 'white', 'red']
}, 'NBUI ajustado');

// Umbral para definir zonas urbanas (NBUI > 0.2 por ejemplo)
var zonasUrbanas = nbuiMasked.gt(0.2).selfMask(); // 1 = urbana, else = null
Map.addLayer(zonasUrbanas, {palette: ['red']}, 'Zonas urbanas');

// Exportar solo zonas urbanas
Export.image.toDrive({
  image: zonasUrbanas,
  description: 'ZonasUrbanas_Lima',
  folder: 'EarthEngine_Exports',
  fileNamePrefix: 'Zonas_Urbanas_Lima_2024',
  region: roi,
  scale: 10,
  crs: 'EPSG:32718',
  maxPixels: 1e13
});


## PRUEBA 1: NDBI

// ROI y colección
var roi = geometry;
var image = ee.ImageCollection("LANDSAT/LC08/C02/T1_L2")
  .filterBounds(roi)
  .filterDate('2024-01-01', '2024-02-29')
  .filter(ee.Filter.lt('CLOUD_COVER', 10))
  .median();

// Bandas
var NIR = image.select('SR_B5');   // B5 - NIR
var SWIR1 = image.select('SR_B6'); // B6 - SWIR1

// Calcular NDBI
var ndbi = SWIR1.subtract(NIR)
  .divide(SWIR1.add(NIR))
  .rename('NDBI');

// Recortar al ROI
var ndbi_clip = ndbi.clip(roi);

// Visualización detallada
Map.centerObject(roi, 12);
Map.addLayer(ndbi_clip, {
  min: -0.5,
  max: 0.5,
  palette: ['blue', 'white', 'red']
}, 'NDBI estilo completo');

// Umbral para zonas urbanas
var urbanNDBI = ndbi_clip.gt(0.1).selfMask(); // puedes ajustar el 0.1
Map.addLayer(urbanNDBI, {palette: ['red']}, 'Zonas urbanas (NDBI)');

// Exportar
Export.image.toDrive({
  image: urbanNDBI,
  description: 'ZonasUrbanas_NDBI_Lima',
  folder: 'EarthEngine_Exports',
  fileNamePrefix: 'NDBI_Zonas_Urbanas_Lima_2024',
  region: roi,
  scale: 30,
  crs: 'EPSG:32718',
  maxPixels: 1e13
});





## PRUEBA 1: UI

// ROI y colección
var roi = geometry;
var image = ee.ImageCollection("LANDSAT/LC08/C02/T1_L2")
  .filterBounds(roi)
  .filterDate('2024-01-01', '2024-02-29')
  .filter(ee.Filter.lt('CLOUD_COVER', 10))
  .median();

// Bandas necesarias
var SWIR2 = image.select('SR_B7'); // B7 - SWIR2
var RED = image.select('SR_B4');   // B4 - Red

// Calcular UI (Urban Index)
var ui = SWIR2.subtract(RED)
  .divide(SWIR2.add(RED))
  .rename('UI');

// Recortar al ROI
var ui_clip = ui.clip(roi);

// Visualización del índice
Map.centerObject(roi, 12);
Map.addLayer(ui_clip, {
  min: -0.5,
  max: 0.5,
  palette: ['blue', 'white', 'red']
}, 'Urban Index (UI)');

// Umbral para zonas urbanas
var urbanUI = ui_clip.gt(0.1).selfMask(); // ajusta el umbral si es necesario
Map.addLayer(urbanUI, {palette: ['red']}, 'Zonas urbanas (UI)');

// Exportar
Export.image.toDrive({
  image: urbanUI,
  description: 'ZonasUrbanas_UI_Lima',
  folder: 'EarthEngine_Exports',
  fileNamePrefix: 'UI_Zonas_Urbanas_Lima_2024',
  region: roi,
  scale: 30,
  crs: 'EPSG:32718',
  maxPixels: 1e13
});




## PRUEBA 1: NBUI SENTINEL

PRUEBA 2: SATELITE SENTINEL

var roi = geometry;
// PRUEBA 1
// Colección Sentinel-2 SR (superficie reflectancia)
var collection = ee.ImageCollection("COPERNICUS/S2_SR_HARMONIZED")
  .filterBounds(roi)
  .filterDate('2024-01-01', '2024-02-29')
  .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 10));

// Función para enmascarar nubes usando QA60
function maskS2clouds(image) {
  var qa = image.select('QA60');
  var cloudBitMask = 1 << 10;
  var cirrusBitMask = 1 << 11;
  var mask = qa.bitwiseAnd(cloudBitMask).eq(0)
               .and(qa.bitwiseAnd(cirrusBitMask).eq(0));
  return image.updateMask(mask).divide(10000);  // escalar reflectancia
}

// Imagen compuesta libre de nubes
var image = collection.map(maskS2clouds).median().clip(roi);

// Selección de bandas adaptadas
var B2 = image.select('B3');   // Verde
var B3 = image.select('B4');   // Rojo
var B4 = image.select('B8');   // NIR
var B5 = image.select('B11');  // SWIR1
var B6 = image.select('B8');   // Sustituto aproximado para térmica (no hay en Sentinel)

// Parámetro I
var I = ee.Number(1);

// NBUI adaptado
var term1 = B5.subtract(B4).divide(
  B5.add(B6).sqrt().multiply(10)
);

var term2 = B4.subtract(B3).multiply(I.add(1))
  .divide(B4.subtract(B3).add(1));

var term3 = B2.subtract(B5).divide(B2.add(B5));

var nbui = term1.subtract(term2.add(term3)).rename('NBUI');

// NDVI para enmascarar zonas verdes
var ndvi = image.normalizedDifference(['B8', 'B4']); // NDVI = (NIR - RED)/(NIR + RED)
var mask = ndvi.gt(0.09);
var nbuiMasked = nbui.updateMask(mask).clip(roi);

// Visualizar
Map.centerObject(roi, 10);
Map.addLayer(nbuiMasked, {min: -0.5, max: 0.5, palette: ['blue', 'white', 'red']}, 'NBUI ajustado');

// Umbral para zonas urbanas
var zonasUrbanas = nbuiMasked.gt(0.3).selfMask();
Map.addLayer(zonasUrbanas, {palette: ['red']}, 'Zonas urbanas');

// Exportar
Export.image.toDrive({
  image: zonasUrbanas,
  description: 'ZonasUrbanas_Sentinel2',
  folder: 'EarthEngine_Exports',
  fileNamePrefix: 'Zonas_Urbanas_Sentinel2_2024',
  region: roi,
  scale: 10,
  crs: 'EPSG:32718',
  maxPixels: 1e13
});


