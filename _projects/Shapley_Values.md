---
title: "Shapley values as a tool for interpreting machine learning model outcomes"
layout: single
classes: wide
excerpt: "Shapley values can be a powerful technique for uncovering meaning behind machine learning models"
header:
  teaser: /assets/images/Top3_SHAP_boreal_allPN_new.png
  overlay_image: /assets/images/python.png
  overlay_filter: rgba(0, 0, 0, 0.5)
author_profile: true
permalink: /projects/Shapley_Values/
toc: true
---
## What are Shapley values?
Shapley values, coined by [Lloyd Shapley](https://apps.dtic.mil/sti/tr/pdf/AD0604084.pdf), are based on cooperative game theory and are used to calculate the payout of a player based on their contribution to the total payout of the game while considering interactions among the players. In other words, Shapley values are the average expected marginal contribution of one player after all possible combinations have been considered. 

They are particularly useful in cases where contributions by players are unequal. In this way, Shapley values are great for many applications such as business, marketing or in the example I provide later, for interpreting machine learning model outcomes.

## How can Shapley values be used for explaining machine learning model outcomes?
In the case of machine learning, the "game" would be the prediction task for one instances of the dataset, the "players" would be the features or predictor variables and the "total payout" is the predicition made.

### Example of Shapley values being used to explain random forest model results
For example, I built a random forest regression model that predicted vegetation density (basal area - BA) for different tree species across boreal Alaska from 2000 to 2024. These models showed that from 2000 to 2024, black spruce gradually lost density while Alaska birch gradually gained denisty (Fig. 1). White spruce and quaking aspen had more variable trends in density throughout the last 25 years. These patterns were interesting as common thought in boreal Alaska is that conifers (black spruce and white spruce) are decreasing and deciduous hardwood species (Alaska birch and quaking aspen) are increasing due to climate change and its impact on the wildfire regimes.
![Average change in species density from 2000 to 2024](https://hana-matsumoto.github.io/assets/images/BA_timeseries_boreal_smooth.png "Average change in species density from 2000 to 2024")
*Figure 1: Average change in species density from 2000 to 2024*

Therefore, I was interested in determining the contributions of the different predictor variables in predicting outcomes that resulted in losses and gains of vegetation density. To do this I determined sites that lost density vs. gained density from 2000 to 2024 and then calculated Shapley values using the *'iml'* package in R for each of those sites in 2000 and 2024. Using *'ggplot2'*, I created a popular Shapley graph known as a beeswarm graph to illustrate my results.

Beeswarm graphs are packed with a ton of information, revealing not just the relative importance of features, but their actual relationships with the predicted outcome. We can see how these types of graphs work in relation to my example below.
![Shapley values figure explained](https://hana-matsumoto.github.io/assets/images/shapley_val_explained.png "How to read a Shapley value graph")


From this graph, I was able to make a couple of different observations about what drove vegetation density predictions in 2000 vs 2024 and how that may have led to the observed patterns in vegetation density (Fig. 1). For this example, I am only going to explain how these graphs gave insights to the density patterns for black spruce vs. Alaska birch.
![Shapley values figure](https://hana-matsumoto.github.io/assets/images/Top3_SHAP_boreal_allPN_new.png "Top three Shapley values for predicting species basal area")


## References
https://christophm.github.io/interpretable-ml-book/shapley.html
https://shap.readthedocs.io/en/latest/example_notebooks/overviews/An%20introduction%20to%20explainable%20AI%20with%20Shapley%20values.html#
https://www.aidancooper.co.uk/a-non-technical-guide-to-interpreting-shap-analyses/?xgtab&
Shapley, Lloyd S. 1953. “A Value for n-Person Games.” In Contributions to the Theory of Games Ii, edited by Harold W. Kuhn and Albert W. Tucker, 307–17. Princeton: Princeton University Press.
