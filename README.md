# NO2-GEE-Geomundus-2020-workshop

***This repository is aimed to help the students that attended the Geomundus 2020 NO2 mapping workshop***

***It is a Google Earth Engine code to learn how to access and process NO data from TROPOMI Sentinel 5 sensor:***

***To access the GEE JavaScript code console: https://code.earthengine.google.com/***


***Some References:***<br>
https://directory.eoportal.org/web/eoportal/satellite-missions/c-missions/copernicus-sentinel-5p

https://www.bloomberg.com/graphics/2020-how-coronavirus-impacts-climate-change/

https://www.thelancet.com/journals/lanplh/article/PIIS2542-5196(20)30107-8/fulltext

https://esamultimedia.esa.int/multimedia/publications/BR-319/BR319.pdf

https://www.who.int/news-room/fact-sheets/detail/ambient-(outdoor)-air-quality-and-health

https://www.nasa.gov/feature/goddard/2020/nasa-model-reveals-how-much-covid-related-pollution-levels-deviated-from-the-norm

https://earthobservatory.nasa.gov/images/146362/airborne-nitrogen-dioxide-plummets-over-china

https://gmao.gsfc.nasa.gov/research/science_snapshots/2020/air_pollution_COVID-19.php
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

**#REDUCING THE REGION TO GET A VALUE. THE REGION PARAMETER IS THE FEATURE GEOMETRY:** 

**##(CALCULATING THE MIN AND MAX VALUES OF NO2 IN THE AGGREGATED TIME SERIES IN SPMR)**

var max_no2_2019 = no2_jun2019.reduceRegion({<br>
  reducer: ee.Reducer.max(),<br>
  geometry: SPMR.geometry(),<br>
  scale: 30,<br>
  maxPixels: 1e9<br>
});      

print(max_no2_2019, 'max NO2 2019');

var min_no2_2019 = no2_jun2019.reduceRegion({<br>
  reducer: ee.Reducer.min(),<br>
  geometry: SPMR.geometry(),<br>
  scale: 30,<br>
  maxPixels: 1e9<br>
});      

print(min_no2_2019, 'min NO2 2019');


**#USE THE MAX AND MIN VALUES TO CREATE A CUSTOM PALLETE AND VIEW**

var band_viz_no2_2019 = {<br>
  min: 7.21023297316043,<br>
  max: 279.1813350315296,<br>
  palette: ['black', 'blue', 'purple', 'green', 'yellow', 'red']<br>
};

Map.addLayer(no2_jun2019, band_viz_no2_2019, 'NO2 jun/2019');

Map.centerObject(SPMR);

**#LET'S MAKE THE THINGS SMOOTHER! CREATE A KERNEL TO CONVOLVE THE IMAGE, WORKING AS A LOW PASS FILTER**
var boxcar = ee.Kernel.square({<br>
  radius: 10,<br>
  units: 'pixels',<br> 
  normalize: true<br>
});

var no2_2019_conv =  no2_jun2019.convolve(boxcar);

Map.addLayer(no2_2019_conv, band_viz_no2_2019, 'NO2 jun/2019 (convolved)');


**#REPEATING THE PROCESS FOR JUNE/2020 (SORT OF LOCKDOWN TIME IN SÃO PAULO!)**<br>
**##ADD A NO2 COLLECTION FOR JUNE/2020**

var no2_jun2020 = ee.ImageCollection("COPERNICUS/S5P/NRTI/L3_NO2")

            .filterDate('2020-06-01', '2020-06-30')
            .select('tropospheric_NO2_column_number_density')
            .mean()
            .multiply(1000000)
            .clip(SPMR);
            
print(no2_jun2020, 'NO2 2020');


**#REDUCING THE REGION TO GET A VALUE. THE REGION PARAMETER IS THE FEATURE GEOMETRY:**<br> 
**##(CALCULATING THE MIN AND MAX VALUES OF NO2 IN THE AGGREGATED TIME SERIES IN SPMR)**

var max_no2_2020 = no2_jun2020.reduceRegion({<br>
  reducer: ee.Reducer.max(),<br>
  geometry: SPMR.geometry(),<br>
  scale: 30,<br>
  maxPixels: 1e9<br>
});      

print(max_no2_2020,'max NO2 2020');

var min_no2_2020 = no2_jun2020.reduceRegion({<br>
  reducer: ee.Reducer.min(),<br>
  geometry: SPMR.geometry(),<br>
  scale: 30,<br>
  maxPixels: 1e9<br>
});      

print(min_no2_2020, 'min NO2 2020');

Map.addLayer(no2_jun2020, band_viz_no2_2019, 'NO2 jun/2020');<br>
Map.centerObject(SPMR);

var no2_2020_conv =  no2_jun2020.convolve(boxcar);

**#NOTICE THAT WE ARE USING THE SAME MIN AND MAX SCALE FROM 2019, IN ORDER TO COMPARE BOTH**

Map.addLayer(no2_2020_conv, band_viz_no2_2019, 'NO2 jun/2020 (convolved)');

**#EXPORTING THE MAP (TIFF IMAGE) TO YOUR CLOUD STORAGE**

Export.image.toDrive({<br>
  image: no2_2020_conv,<br>
  description: 'no2_2020_conv',<br>
  scale: 100<br>
});


**#CHARTS ANDA DATA AGGREGATION (INCLUDING EXPORT!)**

**##LET'S CREATE SOME NEW FILTERS TO COMPARE OVER YEARS IN THE SAME PERIODS**


**#Add a NO2 from January to December 2019

var no2_chart_2019 = ee.ImageCollection("COPERNICUS/S5P/NRTI/L3_NO2")

            .filterDate('2019-01-01', '2019-12-31')
            .select('tropospheric_NO2_column_number_density')
            .filterBounds(SPMR);
            
            
print(no2_chart_2019, 'NO2 2019 filter');    


var no2_2019 = ui.Chart.image.doySeries(<br>
    no2_chart_2019, sp_city, ee.Reducer.mean(), 500, ee.Reducer.mean());
    
    
print(no2_2019, 'NO2 by DOY - 2019 - São Paulo city');


**#Now, let's compare 2019 and 2020 NO2 concentrations in the same chart**

var no2_ByYear = ee.ImageCollection("COPERNICUS/S5P/NRTI/L3_NO2")

    .filterDate(ee.Date('2019-01-01'), ee.Date('2020-10-31'))
    .select('tropospheric_NO2_column_number_density')
    .filterBounds(SPMR);

var no2_19_20 = ui.Chart.image.doySeriesByYear(<br>
    no2_ByYear, 'tropospheric_NO2_column_number_density', sp_city, ee.Reducer.mean(), 500, ee.Reducer.mean());
    
print(no2_19_20, 'NO2 by DOY - 2019 versus 2020 - São Paulo city');    


**#Define a chart with different series for each region, averaged by DOY.**

var region_comp = ee.ImageCollection("COPERNICUS/S5P/NRTI/L3_NO2")

    .filterDate(ee.Date('2020-04-01'), ee.Date('2020-04-30'))
    .select('tropospheric_NO2_column_number_density')
    .filterBounds(SPMR);
    
print(region_comp, 'Regions comparisson');

var multiply = function(img){<br>
  return img.multiply(1000000).copyProperties(img, ['system:time_start']);<br>
};

var no2_comp_mult = region_comp.map(multiply);

print(no2_comp_mult, 'no2 in micro moles');

var sp_x_ssa = ui.Chart.image.doySeriesByRegion(<br>
    no2_comp_mult, 'tropospheric_NO2_column_number_density', sp_ssa, ee.Reducer.mean(), 500, ee.Reducer.mean(), 'city');

print(sp_x_ssa, 'NO2: São Paulo versus Salvador de Bahia - April/2020');


**#Using a similar approach with a different chart method: image.series**

var collection = ee.ImageCollection('COPERNICUS/S5P/NRTI/L3_NO2')

.select('NO2_column_number_density')
.filterDate('2019-09-01', '2020-09-30')
.filterBounds(SPMR);


var mean=ui.Chart.image.series(collection, sp_city, ee.Reducer.mean(), 500).setOptions({title: 'NO2 mean'});
var max=ui.Chart.image.series(collection, sp_city, ee.Reducer.max(), 500).setOptions({title: 'NO2 max'});
var min=ui.Chart.image.series(collection, sp_city, ee.Reducer.min(), 500).setOptions({title: 'NO2 min'});


print(mean, 'mean image series chart');
print(max, 'max image series chart' );
print(min, 'min image series chart');


**###(BONUS!) ADD A CO2 COLLECTION###**

var co2_sep2020 = ee.ImageCollection("COPERNICUS/S5P/OFFL/L3_CO")

            .filterDate('2020-09-01', '2020-09-30')
            .select('CO_column_number_density')
            .mean()
            .clip(BR);
            
var co2_sep2019 = ee.ImageCollection("COPERNICUS/S5P/OFFL/L3_CO")

            .filterDate('2019-09-01', '2019-09-30')
            .select('CO_column_number_density')
            .mean()
            .clip(BR);           
            
var band_viz_co2 = {<br>
  min: 0.020,<br>
  max: 0.086,<br>
  palette: ['black', 'blue', 'purple', 'cyan', 'green', 'yellow', 'red']<br>
};

Map.addLayer(co2_sep2020, band_viz_co2, 'co2 2020');
Map.addLayer(co2_sep2019, band_viz_co2, 'co2 2019');

Map.centerObject(BR);



