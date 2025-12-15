---
title: "Everest Base Camp"
layout: single
classes: wide
excerpt: "Map of a popular hike to the Mt. Everest south base camp"
header:
  overlay_image: /assets/images/EverestBaseCamp.jpg
  overlay_filter: rgba(0, 0, 0, 0.5)
  teaser: /assets/images/EverestBaseCamp.jpg
permalink: /maps/everest-basecamp/
author_profile: true
---
Mt. Everest south base camp, Nepal

<!-- Static image -->
<img
  src="/assets/images/EverestBaseCamp.jpg"
  alt="Everest South Base Camp map"
  style="cursor: zoom-in; max-width:100%;"
  onclick="openEverestMap()">


<!-- Fullscreen modal -->
<div id="everest-modal"
     style="
       display:none;
       position:fixed;
       top:0; left:0;
       width:100%; height:100%;
       background:rgba(0,0,0,0.85);
       z-index:9999;
     "
     onclick="closeEverestMap()">

  <!-- OpenSeadragon viewer -->
  <div id="everest-viewer"
       style="width:100%; height:100%;"
       onclick="event.stopPropagation()">
  </div>

  <!-- Close button -->
  <button onclick="closeEverestMap()"
          style="
            position:absolute;
            top:15px; right:20px;
            font-size:32px;
            color:white;
            background:none;
            border:none;
            cursor:pointer;
          ">
    &times;
  </button>
</div>


<script>
let everestViewer = null;

function openEverestMap() {
  document.getElementById("everest-modal").style.display = "block";

  if (!everestViewer) {
    everestViewer = OpenSeadragon({
      id: "everest-viewer",
      prefixUrl: "https://cdnjs.cloudflare.com/ajax/libs/openseadragon/4.1.0/images/",
      tileSources: {
        type: "image",
        url: "/assets/images/EverestBaseCamp.jpg"
      },
      showNavigator: true,
      defaultZoomLevel: 1,
      maxZoomPixelRatio: 2
    });
  }
}

function closeEverestMap() {
  document.getElementById("everest-modal").style.display = "none";
}

/* Close on ESC key */
document.addEventListener("keydown", function (e) {
  if (e.key === "Escape") closeEverestMap();
});
</script>


For my first successful 3D map, I designed this map for my friend for her birthday as she was hiking this trail up to the Mt. Everest south base camp as birthday present to herself (hi Liza). 

I used QGIS, Blender and Adobe Illustrator and PhotoShop to create this map. The basemap I used is actually an antique map produced by the Royal Geographical Society in cooperation with the Mount Everest Foundation and archived at [RareMaps](https://www.raremaps.com/gallery/detail/95522/the-mount-everest-region-royal-geographical-society) and [University of Wisconsin Milwaukee's Digital Map Collection library](https://collections.lib.uwm.edu/digital/collection/agdm/id/30615/rec/2). I also considered using other maps from the [David Rumsey Map Collection](https://www.davidrumsey.com/luna/servlet/view/search?q=mt+everest&annotSearch=) which is another great resource for antique maps. The trail data I was able to find from a user on [Google My Maps](https://www.google.com/maps/d/u/0/viewer?mid=1R6MkAXWvhvvywV6huHq0AnoEeWPkxvNI&ll=27.855285993962884%2C86.803691868863&z=11). I downloaded this as a KML and edited it using both ArcGIS Pro and later PhotoShop. The basemap along with the trail was place ontop a 90 m Copernicus DEM - Global and European Digital Elevation Model in Blender.

For my first 3D map, I was happy with how it turned out. However, I wish I would have rescaled the DEM to get rid of some of the boxy-ness/pixelation of the map. This is something I improved upon in my map of Emma Lake.
