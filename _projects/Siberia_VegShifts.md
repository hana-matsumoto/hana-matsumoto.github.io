---
title: "What simulation modeling and random forest models can tell us about the future of Siberian taiga"
layout: single
classes: wide
excerpt: "Analyzing simulation model output using random forest models and variable importance to investigate future changes in Siberian forests."
header:
  teaser: /assets/images/Top3Vars_noncond.png
  overlay_image: /assets/images/python.png
  overlay_filter: rgba(0, 0, 0, 0.5)
permalink: /projects/Siberia_VegShifts/
author_profile: true
toc: true
---
The rapid warming of high-latitude ecosystems poses a significant risk to the world's largest terrestrial carbon stock in boreal forests. This research project addresses the critical need to understand how these ecosystems will respond to climate change. 

The primary goal of this project was to **simulate the complex vegetation dynamics of Siberian taiga 285 years in the future and parse out the important drivers of any simulated changes using predictive models**.

# Methods
## Simulation modeling
This study utilized the [LANDIS-II forest landscape model](https://www.landis-ii.org/) to simulate forest dynamics 285 years into the future in Siberia. Four different study landscapes called the tundra, northern taiga, middle taiga and southern taiga (Fig. 1) were defined based on the distinct differences in vegetation composition between the four landscapes.
![](https://hana-matsumoto.github.io/assets/images/Figure1.png)

These landscapes were simulated under three climate scenarios including a historical baseline scenario (little to no climate change), an intermediate climate change scenario and an extreme climate chanage scenario (Fig. 2) and exposed to three simulated disturbances; wildfire, wind, and timber harvesting.
![](https://hana-matsumoto.github.io/assets/images/Figure2.png)

## Analysis
Analysis to summarize how the landscapes in Siberia changed in forest composition over the simulation period was conducted (Fig. 3). From this analysis, the primary simulated vegetation changes from each landscape were further investiagted using random forest models to determine the potential drivers behind such changes.

### Random forest classification models
I developed and implemented a series of 48 random forest classification models using the ***'tidymodels'*** and ***'ranger'*** packages in R. Response variables differed between landscapes depending on the primary simulated vegetation change of interest. The model's predictive variables included simulated climate data, disturbance (fire, wind, timber harvesting), and topographical variables. 

| Category | Variable name |
| ------- | -------------- |
| Response variable | Tundra: Arctic grass/moss retention or transition to conifer |
| | Taigas: conifer retention or transition to deciduous hardwood |
| -------------- | -------------------------------------- |
| Predictor variable | Mean seasonal temperature post-disturbance |
|                    | Year-to-year variance in mean seasonal temperature post-disturbance |
|                    | Change in mean annual seasonal temperature |
|                    | Mean total seasonal precipitation |
|                    | Variance in total annual precipitation |
|                    | Change in mean total seasonal precipitation |
|                    | Mean daily summer PAR |
|                    | Aspect |
|                    | Slope |
|                    | Disatnce to closest transitional seed source |
|                    | Average soil water content over recovery period |
|                    | Change in soil water content over recovery period |
|                    | Ecoregion |
|                    | Disturbance type: wildfire, wind, and/or timber harvesting |
|                    | Time since disturbance |


1A key part of this process involved **feature engineering to translate these ecological variables into a format suitable for the models**. The performance of these predictive models was rigorously evaluated using AUC-ROC and accuracy metrics and importance values were extracted from each trained model. These importance values served as the interpretative method for deducing drivers of vegetation change.

# Results
## Simulation modeling results
![](https://hana-matsumoto.github.io/assets/images/Figure4.png)

## Random forest classification model results
The Random Forest models accurately predicted vegetation composition and species distribution with high performance metrics (average accuracy over 82% and AUC-ROC over 90%). The model's variable importance analysis (Fig. 4) revealed that wildfire was the most significant driver of change, with fire activity projected to nearly double under warming scenarios. The models also quantified the projected effects of climate change on commercially valuable timber, showing significant increases in harvestable areas in the tundra (3,802% increase) and a decrease in the southern taiga (54-66% decrease), as detailed in Figures 11 and 12.

# Conclusions
The findings demonstrate the effectiveness of using a data-driven machine learning approach to model and predict complex ecological systems. This research provides valuable, quantifiable insights for environmental management and resource forecasting by identifying the key drivers of change and predicting their impact on boreal forests at a landscape scale.
