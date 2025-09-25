---
title: "What simulation modeling and random forest models can tell us about the future of Siberian taiga"
layout: single
classes: wide
excerpt: "Analyzing simulation model output using random forest models and variable importance to investigate future changes in Siberian forests"
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
*Figure 1: Study landscapes*

These landscapes were simulated under three climate scenarios including a historical baseline scenario (little to no climate change), an intermediate climate change scenario and an extreme climate chanage scenario (Fig. 2) and exposed to three simulated disturbances; wildfire, wind, and timber harvesting.
![](https://hana-matsumoto.github.io/assets/images/Figure2.png)
*Figure 2: Differences between climate scenarios*

## Analysis
Analysis to summarize how the landscapes in Siberia changed in forest composition over the simulation period was conducted (Fig. 3). From this analysis, the primary simulated vegetation changes from each landscape were further investiagted using random forest models to determine the potential drivers behind such changes.

### Random forest classification models
I developed and implemented a series of 48 random forest classification models using the ***'tidymodels'*** and ***'ranger'*** packages in R. Response variables differed between landscapes depending on the primary simulated vegetation change of interest. The model's predictive variables included simulated climate data, disturbance (fire, wind, timber harvesting), and topographical variables. 

| Category | Variable name |
| ------- | -------------- |
| Response variable | Tundra: Arctic grass/moss retention or transition to conifer |
| | Taigas: conifer retention or transition to deciduous hardwood |
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


A key part of this process involved **feature engineering to translate these ecological variables into a format suitable for the models**. The performance of these predictive models was rigorously evaluated using AUC-ROC and accuracy metrics and importance values were extracted from each trained model. These importance values served as the interpretative method for deducing the potential drivers of vegetation change observed in the simulation models.

# Results
## Simulation modeling results
The primary shifts of vegetation that occurred under climate change conditions was that of the rather substantial shift from a **tundra dominated historically by Arctic moss and grass (light blue)** to a tundra **invaded and taken over by conifer species (dark and light green) under intermediate and extreme climate change conditions**. 

In the **three taiga landscapes**, the primary shift of vegetation was that of the **reduction of the historically dominant conifer species (dark and light green)** to landscapes **more heavily dominated by deciduous species (brown) under intermediate and extreme climate change conditions**.
![](https://hana-matsumoto.github.io/assets/images/Figure4.png)
*Figure 3: Changes in percent of vegetation types across the four landscapes for a 285-year simulation period, under a historical climate, intermediate and extreme climate change scenario. Percentages based on the number of hectares of each forest type*

## Random forest classification model results
### Model performance

|               | Historical       |              | Intermediate     |              | Extreme         |              |
|---------------|------------------|--------------|------------------|--------------|-----------------|--------------|
|               | R<sup>2</sup>    | AUC-ROC      | R<sup>2</sup>    | AUC-ROC      | R<sup>2</sup>   | AUC-ROC      |
| Tundra        | 0.99             | 0.99        | 0.94             | 0.98         | 0.99            | 0.99       |
| Northern taiga| 0.97             | 0.96        | 0.68             | 0.74         | 0.69            | 0.75        |
| Middle taiga  | 0.82             | 0.90        | 0.68             | 0.77         | 0.68            | 0.75        |
| Southern taiga| 0.77             | 0.86        | 0.67             | 0.75         | 0.74            | 0.80        |

The random forest models accurately predicted vegetation composition and species distribution with high performance metrics (average accuracy - R<sup>2</sup> over 80% and AUC-ROC over 85%). 

### Variable importance
The model's variable importance analysis (Fig. 4) revealed that two common predictor variables, distance to seed source (SeedDist - green) and soil dynamics (AvgWater, AvgALD, ChangeWater - brown), became increasingly important or became more similarly important to each other for predicting shifts of vegetation in all of the landscapes under climate change conditions. An exception can be seen in the tundra where variables related to climate (Summer_AvgPAR, Summer_ppt, Winter_varppt - yellow) became more important for predicting vegetation shifts under climate change conditions.
![](https://hana-matsumoto.github.io/assets/images/Top3Vars_noncond.png)
*Figure 4: Top three most important variables for classifying vegetation determined by their average variable importance*

If we take a closer look at these top important variables through time, we can uncover more information about what drove these simulated shifts in vegetation. For this example, we are only going to focus on the tundra and southern taiga landscapes.


![](https://hana-matsumoto.github.io/assets/images/all_time_vars_tundra.png)
*Figure 5: Temporal trends of top important variables for the tundra*

![](https://hana-matsumoto.github.io/assets/images/fire_tundra.png)
*Figure 6: Total area burned and area burned by high severity fire over the 285 year simulation period for the tundra*

![](https://hana-matsumoto.github.io/assets/images/all_time_vars_staiga.png)

![](https://hana-matsumoto.github.io/assets/images/fire_staiga.png)

# Conclusions
The findings demonstrate the effectiveness of using a data-driven machine learning approach to model and predict complex ecological systems. This research provides valuable, quantifiable insights for environmental management and resource forecasting by identifying the key drivers of change and predicting their impact on boreal forests at a landscape scale.
