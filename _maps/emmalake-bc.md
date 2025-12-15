---
title: "Emma Lake, British Columbia"
layout: single
classes: wide
excerpt: "Fun 3D map of Emma Lake"
header:
  teaser: /assets/images/EmmaLake_BC.png
  overlay_image: /assets/images/EmmaLake_BC.png
  overlay_filter: rgba(0, 0, 0, 0.5)
permalink: /maps/emmalake-bc/
author_profile: true
---
Emma Lake in British Columbia, Canada
[![Emma Lake in British Columbia](/assets/images/EmmaLake_BC.png "Emma Lake in British Columbia")](/assets/images/EmmaLake_BC.png "Emma Lake in British Columbia"){: .image-popup }

I created this map for my friend's partner's birthday (hi Alex and Sam). Emma Lake is a special place, nestled in the coast range of British Columbia near Powell River. Armed wtih the location from my friend and some text she wanted inlcuded, I came up with this design.

To create this map I developed a true color composite image using 10 m [Sentinel imagery](https://browser.dataspace.copernicus.eu/?zoom=5&lat=50.16282&lng=20.78613&demSource3D=%22MAPZEN%22&cloudCoverage=30&dateMode=SINGLE) in ArcGIS Pro. I overlaid this composite on top of a 30 m DEM from Natural Resources Canada's [Medium Resolution Digital Elevation Model of Canada](https://open.canada.ca/data/en/dataset/18752265-bda3-498c-a4ba-9dfe68cb98da) in Blender which I rescaled and resampled. Map projection is WGS 1984 UTM Zone 10N.

Something I wish I would have done btter was the rendering of the imagery in Blender. I would have liked the satellite imagery to be a bit more crisp in the final product but couldn't quite figure out what was going wrong in time for when this needed printed. Looking forward to figuring this out in future projects!
