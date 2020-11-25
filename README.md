# NO2-GEE-Geomundus-2020-workshop

***This repository is aimed to help the students that attended the Geomundus 2020 NO2 mapping workshop***

***It is a Google Earth Engine code to learn how to access and process NO data from TROPOMI Sentinel 5 sensor:***
__________________________________________________________________________________________________________________________________

**The example regions that we are going to use in the exercise**

var GHSL = ee.ImageCollection("JRC/GHSL/P2016/POP_GPW_GLOBE_V1");

var BR = ee.FeatureCollection("users/pedrassoli_julio/LIMITS_BR");

var SPMR = ee.FeatureCollection("users/pedrassoli_julio/limites_rmsp");

var SP = ee.FeatureCollection("users/pedrassoli_julio/SP_State_limits");

var sp_city = ee.FeatureCollection("users/pedrassoli_julio/Sao_Paulo_city_limits");

var sp_ssa = ee.FeatureCollection("users/pedrassoli_julio/sp_ssa");

**#ACCESSING NO2 DATA**

**ADD, FILTER AND PROCESS AN IMAGE FROM THE COLLECTION**

var no2 = ee.ImageCollection("COPERNICUS/S5P/NRTI/L3_NO2")

            .filterDate('2018-01-01', '2018-12-31')
            .select('tropospheric_NO2_column_number_density')
            .mean()
            .multiply(1000000)
            .clip(SP);
            
print(no2, 'mean NO2 2018 São Paulo State');

var band_viz_no2 = {
  min: 10,
  max: 255.4,
  palette: ['white', 'blue', 'purple', 'green', 'yellow', 'red']
};

Map.addLayer(no2, band_viz_no2, 'Tropospheric NO2 - São Paulo State');
Map.centerObject(SP);

**#ADDING A POPULATION GRID LAYER FOR COMPARISSON**

**##GET THE MAXIMUM POPULATION COUNT AND CLIP BY A REGION**

var popBR = ee.ImageCollection(GHSL)

            .select('population_count')
            .max()
            .clip(SP);
            
var popVis = {
  min: 0,
  max: 2000,
  palette: ['FFFAFA', '000000', '448564', '70daa4', '83ffbf', 'ffffff'],
};

print(popBR);

Map.addLayer(popBR, popVis);

**#COMPARING NO2 DENSITY BEFORE AND DURING LOCKDOWN**

**##ADD A NO2 COLLECTION FOR JUNE/2019**

var no2_jun2019 = ee.ImageCollection("COPERNICUS/S5P/NRTI/L3_NO2")

            .filterDate('2019-06-01', '2019-06-30')
            .select('tropospheric_NO2_column_number_density')
            .mean()
            .multiply(1000000)
            .clip(SPMR);
            
print(no2_jun2019, 'NO2 2019');

