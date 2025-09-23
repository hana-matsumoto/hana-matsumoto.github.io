---
title: "How can predictive models be used to analyze vegetation changes in Alaska?"
layout: single
classes: wide
excerpt: "Leveraging random forest models and remotely sensed data to investigate how vegetation is changing in boreal Alaska"
header:
  teaser: /assets/images/BA_timeseries_boreal_smooth.png
permalink: /projects/AK_VegShifts/
toc: true
---
This project focused on a predicting species vegetation density, scaling up real-world data using machine learning. The objective was to determine species-level changes of the boreal forests in Alaska from 2000 to 2024 as predicted by the machine learning models.

## Methods
This work leveraged geospatial data, time-series analysis and random forest models to analyze forest change. 

I used the Google Earth Engine platform and its Python API to process and analyze remotely sensed data for a 25-year period to build out predictor variables for my random forest models.
Example code for creating some of the predictor variables from remotely sensed data:
```python
# %% Start session
import ee
import pandas as pd
import geemap
import numpy as np

ee.Authenticate()
ee.Initialize()

# %% Area of interest and topo data
AK_boreal = ee.FeatureCollection('projects/ee-vegshiftsalaska/assets/boreal_AK')
#AK_boreal = ee.FeatureCollection('projects/ee-vegshiftsalaska/assets/Dalton_Landis') 
states = ee.FeatureCollection('TIGER/2016/States') 
AK_landscape = states.filter(ee.Filter.eq('NAME', 'Alaska'))

water_ds = ee.Image("JRC/GSW1_4/GlobalSurfaceWater")
water_mask = water_ds.select('max_extent')
non_water = water_mask.eq(0).reproject(crs='EPSG:3338', scale=30)

# # Turn that raster into a vector of all non-water areas
# non_water_vector = non_water.selfMask().reduceToVectors(
#     geometryType='polygon',
#     reducer=ee.Reducer.countEvery(),
#     scale=30,
#     geometry=AK_landscape.geometry(),  # clip to your area of interest
#     maxPixels=1e10
# )

# # Intersect your feature collection with the non-water area
# masked_features = AK_landscape.map(
#     lambda feature: feature.intersection(non_water_vector.geometry(), ee.ErrorMargin(1))
# )

# AK_land_no_water = masked_features.map(
#     lambda f: f.simplify(100)  # simplify by 100 meters
# )

# Map.addLayer(AK_land_no_water, {}, 'crop')

Map = geemap.Map(center=[64, -152], zoom=5, basemap='Esri.WorldGrayCanvas')

# Topographic data
topo_data = ee.ImageCollection('JAXA/ALOS/AW3D30/V3_2').select('DSM')
elevation = topo_data.mosaic().reproject(crs='EPSG:4326', scale=30).toFloat()
slope = ee.Terrain.slope(topo_data.mosaic().reproject(crs='EPSG:4326', scale=30))
aspect = ee.Terrain.aspect(topo_data.mosaic().reproject(crs='EPSG:4326', scale=30))
topo_layers = elevation.addBands(slope).addBands(aspect)
Map.addLayer(elevation, {}, 'crop')
# Ecoregions
#ecoregions = ee.FeatureCollection('projects/ak-project-438916/assets/AK_ecoregions')
#ecoregions_raster = ee.Image('projects/ak-project-438916/assets/AK_eco_raster')
ecoregions_raster = ee.Image('projects/ak-project-438916/assets/eco_int')
eco_raster = ecoregions_raster.select('b1').rename('ecoregion').toFloat()

#Map.addLayer(ecoregions_raster, {}, 'eco AK')
# Map.addLayer(AK_landscape, {}, 'AK')

# %% time range of interest
#years = ee.List.sequence(2000, 2000)
years = ee.List.sequence(2000, 2017).remove(2016)
year_info = years.size().getInfo()
year_info

#%% rescale from 0-65000 to approx. 0-1
def scale_and_offset(img):
    band_list = ['blue', 'green', 'red', 'nir', 'swir1', 'swir2']
    qa_band = img.select('pixel_qa')
    mult_factor = ee.Image.constant(0.0000275)
    add_factor = ee.Image.constant(-0.2)
    
    scaled_bands = []
    for band in band_list:
        scaled_band = img.select(band).multiply(mult_factor).add(add_factor)
        scaled_bands.append(scaled_band)

    img_scaled = ee.Image(scaled_bands).addBands(qa_band)
    img_scaled = img_scaled.set({
        'SUN_AZIMUTH': img.get('SUN_AZIMUTH'),
        'SUN_ELEVATION': img.get('SUN_ELEVATION')
    })
    return img_scaled

# %% Landsat sensor corrections and cloud masks

def cloud_mask_landsat8(img):
    quality_band = img.select('pixel_qa')
    dilated_cloud = (1 << 1)
    cloud_bit = (1 << 3)
    cloud_shadow = (1 << 4)
    water_bit = (1 << 7)
    cloud_mask = (quality_band.bitwiseAnd(cloud_bit).eq(0)) \
    .And(quality_band.bitwiseAnd(cloud_shadow).eq(0)) \
    .And(quality_band.bitwiseAnd(dilated_cloud).eq(0))

    # select pixels flagged as water
    water_mask = quality_band.bitwiseAnd(water_bit).neq(0)

    # Convert to bands
    cloudM = cloud_mask.select([0], ['cloudM'])
    waterM = water_mask.select([0], ['waterM'])
    image_cloud_masked = img.updateMask(cloud_mask).addBands([cloudM, waterM])
    return image_cloud_masked

#%% Vegetation indices
# normalized difference vegetation index
def ndvi_calc(img):
    ndvi = img.normalizedDifference(['nir', 'red']) \
              .rename('ndvi')
    return ndvi
# enhanced vegetation index
def evi_calc(img):
    # coefficients
    c1 = 2.5
    c2 = 6 
    c3 = 7.5
    c4 = 1.0

    # define bands
    red = img.select('red')
    nir = img.select('nir')
    blue = img.select('blue')

    # equation
    evi = nir.subtract(red) \
        .divide(nir.add(red.multiply(c2)).subtract(blue.multiply(c3)).add(c4)).multiply(c1) \
        .rename('evi')
    return evi
# modified normalized difference water index
def mndwi_calc(img):
    mndwi = img.normalizedDifference(['green', 'swir2']) \
              .rename('mndwi')
    return mndwi

 %% Add indices and band names functions
def add_indices(img):
    ndvi = img.addBands(ndvi_calc(img)).toFloat()
    evi = img.addBands(evi_calc(img)).toFloat()
    mndwi = img.addBands(mndwi_calc(img)).toFloat()

    return img.addBands(ndvi).addBands(evi)\
              .addBands(mndwi).addBands(nbr)\
              .addBands(vari).addBands(savi).addBands(tc)

# add suffix to all band names
def add_suffix(in_image, suffix_str):
    # convert band names to lowercase and add suffix
    def append_suffix(band_name):
        return ee.String(band_name).toLowerCase().cat('_').cat(suffix_str)
    # apply the suffix to all band names
    bandnames = in_image.bandNames().map(append_suffix)
    # the number of bands
    nb = bandnames.size()
    # select bands with the new names
    return in_image.select(ee.List.sequence(0, ee.Number(nb).subtract(1)), bandnames)

#%% Get Collection
# renaming bands
def get_landsat_collection_sr(sensor):
    if sensor in ['LC08', 'LC09']:
        bands = ['SR_B2', 'SR_B3', 'SR_B4', 'SR_B5', 'SR_B6', 'SR_B7', 'QA_PIXEL']
    else:  # for Landsat 5 and 7
        bands = ['SR_B1', 'SR_B2', 'SR_B3', 'SR_B4', 'SR_B5', 'SR_B7', 'QA_PIXEL']
    #sensor = 'LC08'
    band_names_landsat = ['blue', 'green', 'red', 'nir', 'swir1', 'swir2', 'pixel_qa']
    cloud_threshold = 60
    collection_filtered_without_date = ee.ImageCollection('LANDSAT/' + sensor + '/C02/T1_L2') \
        .filterBounds(AK_landscape)\
        .filterMetadata('CLOUD_COVER', 'less_than', cloud_threshold) \
        .select(bands, band_names_landsat)
    return collection_filtered_without_date

# %% Iterate
#year = 2001
for i in range(year_info):
    year = years.get(i).getInfo()
    #mtbs for respective year
    fire_layers = process_mtbs(AK_landscape, year)
    temperature_layer = temperature_seasonal(AK_landscape, year).toFloat()
    precipitation_layer = precipitation_seasonal(AK_landscape, year).toFloat()
    bioclim_layer = worldclim_seasonal(AK_landscape).toFloat()

    #seasonal windows
    startSeason1 = ee.Date.fromYMD(year, 3, 1)
    endSeason1 = ee.Date.fromYMD(year, 5, 31)
    startSeason2 = ee.Date.fromYMD(year, 6, 1)
    endSeason2 = ee.Date.fromYMD(year, 8, 31)
    startSeason3 = ee.Date.fromYMD(year, 9, 1)
    endSeason3 = ee.Date.fromYMD(year, 11, 30)
        
    ################################################
    def get_landsat_Images(sensor, AK_landscape, startSeason, endSeason):  
        collection = get_landsat_collection_sr(sensor)
        cleaned_images = (collection
            .filterBounds(AK_landscape)
            .filterDate(startSeason, endSeason)
            .map(scale_and_offset)
            .map(cloud_mask_landsat8)
            #.map(illuminationCondition)
            #.map(illumination_correction)
            .map(add_indices))
        return cleaned_images

    # function to bring everything together            
    def make_ls(AK_landscape):
        #tcc_bands = ee.List(['red', 'green', 'blue'])
        bands_indices = ee.List(['ndvi', 'evi', 'mndwi'])
        sensors = ['LC08', 'LC09', 'LE07', 'LT05']
        spring_images = ee.ImageCollection([])
        summer_images = ee.ImageCollection([])
        fall_images = ee.ImageCollection([])

        for sensor in sensors:
            spring_images = spring_images.merge(get_landsat_Images(sensor, AK_landscape, startSeason1, endSeason1))
            summer_images = summer_images.merge(get_landsat_Images(sensor, AK_landscape, startSeason2, endSeason2))
            fall_images = fall_images.merge(get_landsat_Images(sensor, AK_landscape, startSeason3, endSeason3))
        
        # Median composites by season
        springtime = add_suffix(spring_images.median().select(bands_indices), 'spring')
        summertime = add_suffix(summer_images.median().select(bands_indices), 'summer')
        falltime = add_suffix(fall_images.median().select(bands_indices), 'fall')

        # add each composite as bands
        return springtime.addBands(summertime).addBands(falltime)

    #make composite
    #tcc_bands_sum = ee.List(['red_summer', 'green_summer', 'blue_summer'])
    landsat_composite = make_ls(AK_landscape)\
        .clip(AK_landscape)\
        .reproject(crs='EPSG:3338', scale=30)\
        .updateMask(non_water)

############## Sampling CAFI plots
    def get_pixel_values(f, img):
        return f.setMulti(img.reduceRegion(
            reducer=ee.Reducer.mean(),
            geometry=f.geometry(),
            scale=30,
            crs='EPSG:3338'))
    
    cafiPlots = ee.FeatureCollection(f'projects/ak-project-438916/assets/Basal_area_{year}')
    def sample_pixels(f):
        return get_pixel_values(f)

    pv_sampling = cafiPlots.map(lambda f: get_pixel_values(f, landsat_clim))
    task_sampling = ee.batch.Export.table.toDrive(
       collection=pv_sampling,
       description=f'allCAFI_pix_notc-{year}',
       folder='AK_proj',
       fileFormat='CSV')
    task_sampling.start()
```

These predictor variables were then used in the training and tuning of random forest models which were used to make spatially complete vegetation density maps in R.
Example R code of training and tuning random forest models:
```r
species = c('BlackSpruce', 'WhiteSpruce', 'QAspen', 'AKBirch')

year_info <- c(2000,2024)
final_metrics <- NULL
vip_metrics <- NULL
#final_moran <- NULL

for (spp in species) {
  
  ecoregion_levels <- data.frame(
  value = c(1, 2, 3, 4, 5, 6, 7, 8, 9, 10,
            11, 12, 13, 14, 15, 16, 17, 18, 19, 20),
  label = c("arc_coast", "arc_foot", "brooks", "AK_forlowup", "highlands", 
            "bottomlands", "yukon", "ogilvie", "subarc_coast", "seward", 
            "ahklum_kil", "bristol", "AKpen", "aleutian", "cook", "AKrange",
            "copper", "wrangell", "pacific", "coast_whem")
  )
  
  severity_levels <- data.frame(
    value = c(0, 1, 2, 3, 4, 5, 6, -9999),
    label = c("background", "unburned_low", "low", "moderate", 
              "high", "inc_green", "non-map", "ND")
  )

  # Apply levels to dataframe
  pv_dataframe_spp <- pv_dataframe[pv_dataframe$Species == spp,] %>%
    dplyr::select(-c(1, 7, 13, 15, 17, 19, 34, 36, 38, 40, 42, 44, 46, 48, 60)) %>%
    drop_na() %>%
    dplyr::rename(
      fall_prec_mm = fall_prec_mm_bio,
      fall_temp_C = fall_temp_C_bio,
      spring_prec_mm = spring_prec_mm_bio,
      spring_temp_C = spring_temp_C_bio,
      summer_prec_mm = summer_prec_mm_bio,
      summer_temp_C = summer_temp_C_bio,
      fall_soil_mm = fall_soil_mm_norm,
      fall_vpd_kpa = fall_vpd_mm_norm,
      spring_soil_mm = spring_soil_mm_norm,
      spring_vpd_kpa = spring_vpd_mm_norm,
      summer_soil_mm = summer_soil_mm_norm,
      summer_vpd_kpa = summer_vpd_mm_norm
    ) %>%
    mutate(
      Severity = factor(Severity, 
                        levels = severity_levels$value, 
                        labels = severity_levels$label),
      ecoregion = factor(ecoregion, 
                         levels = ecoregion_levels$value, 
                         labels = ecoregion_levels$label)
    ) %>%
    drop_na()
  
  # Add lat long columns
  coords <- sub('.*\\[([-0-9.]+),([-0-9.]+)\\].*', '\\1,\\2', pv_dataframe_spp$.geo)
  splitxy <- strsplit(coords, ",")
  x <- as.numeric(sapply(splitxy, `[`, 1))
  y <- as.numeric(sapply(splitxy, `[`, 2))
  pv_dataframe_spp$x <- x
  pv_dataframe_spp$y <- y
  pv_dataframe_spp_fin <- pv_dataframe_spp %>%
   dplyr::select(-49) 

  #check data types of columns
   #str(pv_dataframe_spp_fin)
  points_vect <- vect(pv_dataframe_spp_fin, geom = c("x", "y"), crs = "EPSG:4326")

  # Step 2: Reproject to new CRS (e.g., WGS84)
  points_proj <- project(points_vect, "EPSG:3338")

  # Step 3: Extract new coordinates and combine with original attributes
  coords <- crds(points_proj)
  pv_dataframe_spp_proj <- cbind(as.data.frame(points_proj), y = coords[,1], x = coords[,2])
  #write.csv(pv_dataframe_spp_proj, paste0("Data/Output/GEE/pred_vars_", spp, ".csv"), row.names = F)
  
  #split into training and testing
  set.seed(123)
  #data_sf <- st_as_sf(pv_dataframe_spp_proj, coords = c("x", "y"), crs = "EPSG:3338") # for spatial autocorrelation
  data_split <- initial_split(pv_dataframe_spp_proj, prop = 0.95, strata = BA)
  rm(pv_dataframe_spp, pv_dataframe_spp_fin)
  gc()
  
  train_data <- training(data_split)
  test_data  <- testing(data_split)
  
  # test spatial autocorrelation
  # Ensure your training data is an sf object
  # train_sf <- train_data  # already an sf object with geometry
  # 
  # # Extract coordinates
  # coords <- st_coordinates(train_sf)
  # 
  # coords_jittered <- jitter(st_coordinates(train_sf), factor = 0.0001)
  # 
  # # Use jittered coordinates to build neighbors
  # k_neigh <- knearneigh(coords_jittered, k = 8)
  # nb <- knn2nb(k_neigh)
  # lw <- nb2listw(nb, style = "W")
  # 
  # # Run Moran's I test
  # moran_test <- moran.test(train_sf$BA, lw)
  # moran_test$species <- spp
  # moran_test$time <- "before"
  # 
  # # Prepare sf object
  # sf_data <- train_data  # already sf, EPSG:3338
  # 
  # # Generate spatial blocks
  # set.seed(123)
  # spatial_cv <- cv_spatial(
  #   x = sf_data,
  #   k = 10,                  # Number of folds
  #   size = 50000,            # Spatial block size in meters (adjust for your region)
  #   selection = "random"    # "random", "systematic", or "noncontiguous"
  # )
  # 
  # Assign fold IDs back to the data
  # sf_data$folds <- spatial_cv$folds_ids
  # 
  # Convert into a tidymodels-compatible resampling object
  # v_folds_spatial <- group_vfold_cv(sf_data, group = folds)

  v_folds <- vfold_cv(train_data, v =10, strata = BA)
  # Spatial autocorrelation
  # v_folds <- spatial_block_cv(data_sf, method = "random", cellsize = 100000, v=10)
  # v_folds <- spatial_clustering_cv(data_sf, v=10)
  
  rf_mod <- rand_forest(mtry = tune(), min_n = tune(), trees = tune()) %>%
    set_engine("ranger", importance = 'permutation', write.forest = TRUE) %>%
    set_mode("regression")
  
  # Preprocessing data
  rf_recipe <- recipe(BA ~ ., data = train_data) %>% 
    update_role(PSP, Species, new_role = "ID") %>%
    update_role_requirements(role = "ID", bake = FALSE) %>%
    step_corr(all_numeric_predictors(), threshold = 0.95)
  
  rf_workflow <- workflow() %>% 
    add_model(rf_mod)%>%
    add_recipe(rf_recipe)
  
  # TUNE NEW
  reg_grid <- grid_space_filling(
    mtry(range = c(5, 30)),         # Number of predictors to try at each split
    min_n(range = c(1, 20)),        # Minimum number of observations in a node
    trees(range = c(200, 1000)),    # Number of trees in the forest
    size = 20                       # Number of grid combinations to generate
  )
  
  rf_tune <- tune_grid(
     rf_workflow,
     resamples = v_folds,
     grid = reg_grid,
     metrics = metric_set(yardstick::rmse, yardstick::rsq))
  
  rf_best <- rf_tune %>% 
    select_best(metric = 'rmse')
  
  #last workflow
  final_rf_workflow <- finalize_workflow(rf_workflow, rf_best)
  
  # last fit
  keep_pred <- control_resamples(save_pred = TRUE, save_workflow = TRUE)
  
  final_rf_fit <- final_rf_workflow %>%
    fit_resamples(resamples = v_folds, control = keep_pred, metrics = metric_set(rmse, rsq))
  
  metrics_summary <- collect_metrics(final_rf_fit) %>%
    filter(.metric %in% c("rmse", "rsq")) %>%
    dplyr::select(.metric, mean) %>%
    pivot_wider(names_from = .metric, values_from = mean)
  
  metrics_summary$species <- spp
  
  final_metrics <- rbind(final_metrics, metrics_summary)
  
  # Fit on full dataset - don't use for model accuracy, only for making predictions and
  # assessing feat importance
  final_rf_model <- final_rf_workflow %>% fit(data = pv_dataframe_spp_proj)

  # Global importance
  # Extract the ranger fit object
  ranger_model <- extract_fit_parsnip(final_rf_model)$fit
  saveRDS(ranger_model, paste0("Data/Output/RF/Results/ranger_model_", spp, "_newclim.rds"))
  
  # Get variable importance
  vip_df <- as.data.frame(ranger_model$variable.importance)
  vip_df$variable <- rownames(vip_df)
  colnames(vip_df) <- c("importance", "variable")
  
  vip_df <- vip_df[order(-vip_df$importance), ]
  vip_df$species <- spp
  vip_metrics <- rbind(vip_metrics, vip_df)
  
  #write.csv(final_metrics, "Data/Output/RF/Results/rf_final_mets.csv", row.names = F)
  #write.csv(vip_metrics, "Data/Output/RF/Results/vip_mets.csv", row.names = F)

  for (year in year_info) {
  
    files_rast <- list.files("Data/Output/GEE/Composites/",
                               pattern = glob2rx(paste0("boreal-comp-", year, "-*.tif")),
                               full.names = TRUE)
    
    for (file_path in files_rast) {
      raster_with_latlon <- composite_call(file_path)
      #names(raster_with_latlon)
      
      ecoregion_levels <- data.frame(value = c(1, 2, 3, 4, 5, 6, 7, 8, 9, 10,
                                                 11, 12, 13, 14, 15, 16, 17, 18, 19, 20),
                                       label = c("arc_coast", "arc_foot", "brooks", "AK_forlowup", "highlands",
                                                 "bottomlands", "yukon", "ogilvie", "subarc_coast", "seward",
                                                 "ahklum_kil", "bristol", "AKpen", "aleutian", "cook", "AKrange",
                                                 "copper", "wrangell", "pacific", "coast_whem"))
      levels(raster_with_latlon[["ecoregion"]]) <- ecoregion_levels
  
      # burn severity codes and labels
      severity_levels <- data.frame(value = c(0, 1, 2, 3, 4, 5, 6, -9999),
                                       label = c("background", "unburned_low", "low", "moderate",
                                                 "high", "inc_green", "non-map", "ND"))
      levels(raster_with_latlon[["Severity"]]) <- severity_levels
      
      #chunk the raster
      chunk_n <- 10
      rows <- nrow(raster_with_latlon)
      rows_per_chunk <- ceiling(rows / chunk_n)
      cols <- ncol(raster_with_latlon)
      readStart(raster_with_latlon)

      for (i in 1:chunk_n) {
        start_row <- (i - 1) * rows_per_chunk + 1
        end_row <- min(i * rows_per_chunk, rows)
      
        # Get the Y coordinates for the start and end rows
        y_top <- yFromRow(raster_with_latlon, start_row)
        y_bottom <- yFromRow(raster_with_latlon, end_row)
      
        # Define extent for chunk
        chunk_ext <- ext(raster_with_latlon)[1:2]  # xmin, xmax
        chunk_ext <- ext(chunk_ext[1], chunk_ext[2], y_bottom, y_top)
      
        # Crop to chunk
        chunk_raster <- crop(raster_with_latlon, chunk_ext)
        
        # Convert to dataframe for prediction
        df <- as.data.frame(chunk_raster)
      
        # Predict
        pred <- predict(ranger_model, df)
      
        # Copy structure and insert predicted values
        pred_rast <- chunk_raster[[1]]  # single-layer structure
        values(pred_rast) <- pred$predictions
      
        # Save
        comp_name <- sub(paste0(".*boreal-comp-", year, "-([^.]+)\\.tif$"), "\\1", file_path)
        out_name <- paste0("Data/Output/RF/Maps/new/newest/", spp, "_", year, "_", comp_name, "_chunk_", i, ".tif")
        writeRaster(pred_rast, out_name, overwrite = TRUE)
      
        print(paste("Processed chunk", i))
      
        rm(df, chunk_raster, pred_rast)
        gc()
      }

      readStop(raster_with_latlon)
      
      #writeRaster(preds_rast, paste0("Data/Output/RF/Maps/", year, "_pred_map_newclim", spp, ".tif"), overwrite=TRUE)
      
      }
  }
}# end loop
```
## Results
The time-series analysis (Figure X) revealed that vegetation change was often species-specific, challenging the common ecological assumption that species within a functional group (e.g., all conifers) behave similarly. Black spruce was found to be the primary driver of conifer decline, while Alaska birch drove the increase in deciduous trees. This analysis showed that the model's predictions of forest change at the species level were more accurate than those at the functional group level.

![Alt text for the image](assets/images/image.jpg "Optional Title")

## Conclusions
This research highlights the importance of species-level analysis in ecological modeling. The findings suggest that models that aggregate species into functional groups may miss critical trends and overestimate changes. The use of remote sensing and time-series analysis provided a powerful method for validating complex model outputs and provided a more nuanced understanding of forest dynamics.
