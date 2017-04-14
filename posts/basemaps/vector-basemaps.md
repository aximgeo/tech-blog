# Getting ESRI's Default Vector Basemaps into the Basemap Gallery Widget

Whether you agree with putting a basemap gallery widget into your web mapping application or not, it is one of the most common tasks web developers are asked to perform. At some point in developing spatial web applications you will likely be asked to do this. Fortunately, many APIs come with some type of widget and make this task fairly trivial. The ESRI JavaScript API has had a basemap gallery [widget](https://developers.arcgis.com/javascript/3/jsapi/basemapgallery-amd.html) since the early days but new vector basemaps have increased it's relevance as of late. As such, our applications had a requirement to take all of the latest default basemaps within the JSAPI and put them into the gallery.

Vector tile layers have been around for a while but are [relatively new](https://developers.arcgis.com/javascript/3/jshelp/new_v315.html) in the ESRI JSAPI. Moreover, ESRI now provides a number of vector basemaps available from [ArcGIS Online](http://www.arcgis.com/home/group.html?id=30de8da907d240a0bccd5ad3ff25ef4a) for free that can be added to your application. ESRI makes it very easy to add custom basemaps to an application using either the default [basemaps](https://developers.arcgis.com/javascript/3/jsapi/esri.basemaps-amd.html) collection or the gallery widget itself. However, depending on where you want to put the basemap, the code is a bit different. 

To add a basemap to the default collection, simply create a new basemap object with it's collection of basemap layers and add it.

```javascript
require([
    "esri/basemaps",
    "esri/map"
  ], function (esriBasemaps, Map) {
    esriBasemaps.myBasemap = {
      baseMapLayers: [{
        type: "VectorTile",
        url: "https://www.arcgis.com/sharing/rest/content/items/92c551c9f07b4147846aae273e822714/resources/styles/root.json"
        }
      ],
      thumbnailUrl: "https://www.arcgis.com/sharing/rest/content/items/92c551c9f07b4147846aae273e822714/info/thumbnail/streetnight_thumb_b2.jpg",
      title: "My Basemap"
    };

    var map = new Map("map", {
      basemap: "myBasemap",
      center: [-111.879655861, 40.571338776], // long, lat
      zoom: 13
    });
});
```
To add a basemap to the gallery widget, create a new [Basemap](https://developers.arcgis.com/javascript/3/jsapi/basemap-amd.html) object with it's collection of [BasemapLayers](https://developers.arcgis.com/javascript/3/jsapi/basemaplayer-amd.html) and add it to the gallery's constructor.

```javascript
require([
  "esri/dijit/Basemap",
  "esri/dijit/BasemapLayer",
  "esri/dijit/BasemapGallery" 
], function(Basemap, BasemapLayer, BasemapGallery) {
  var basemaps = [];
  var myBasemap = new Basemap({
    layers: [new BasemapLayer({
      styleUrl: "https://www.arcgis.com/sharing/rest/content/items/92c551c9f07b4147846aae273e822714/resources/styles/root.json",
      type: "VectorTileLayer"
    })],
    id: "myBasemap",
    title: "My Basemap",
    thumbnailUrl: "https://www.arcgis.com/sharing/rest/content/items/92c551c9f07b4147846aae273e822714/info/thumbnail/streetnight_thumb_b2.jpg"
  });
  basemaps.push(myBasemap);

  var gallery = new BasemapGallery({
    map: map,
    basemaps: basemaps
  }, "myDiv");
  gallery.startup();
});
```

The main thing to notice in the above examples is that the *basemap layer* object is different. With the `esriBasemaps` collection, vector tile layers are defined with a `url` property and a `type` property set to *VectorTile*. With the gallery, vector tile layers are defined with a `styleUrl` property and a `type` property set to *VectorTileLayer*. While this difference is [documented](https://developers.arcgis.com/javascript/3/jsapi/basemaplayer-amd.html#basemaplayer1-type) in the API it's not highly visible and easily overlooked.

With the above inconsistencies (`VectorTile` versus `VectorTileLayer` and `url` versus `styleUrl`), simply transferring the default basemaps into the gallery requires a bit of manipulation. Here's what we did to accomplish this using Underscore (you could just as easily use Dojo):

```javascript
require([
  "esri/basemaps",
  "esri/dijit/Basemap",
  "esri/dijit/BasemapLayer",
  "esri/dijit/BasemapGallery" 
], function(esriBasemaps, Basemap, BasemapLayer, BasemapGallery) {
  ...
  // don't show ArcGIS basemaps since we'll add those from the default collection
  var basemapGalleryParams = {
    showArcGISBasemaps: false,
    map: map
  };

  // create a new basemaps array containing the default basemaps
  var basemaps = [];
  _.each(_.keys(esriBasemaps), function (key) {
    var basemap = esriBasemaps[key];
    basemaps.push(new Basemap({
      id: key,
      layers: _.map(basemap.baseMapLayers || [], function (l) {
          return l.type && l.type === "VectorTile" ? _.extend(l, {
              type: l.type + "Layer",
              styleUrl: l.url
          }) : l;
      }),
      thumbnailUrl: basemap.thumbnailUrl,
      title: basemap["title"]
    }));
  });

  basemapGalleryParams.basemaps = basemaps;
  var gallery = new BasemapGallery(basemapGalleryParams, "myDiv");
  gallery.startup();
  ...
});
```

And that's it...you now have the entire collection of default basemaps in the widget for use in your application. Perhaps in the future, ESRI will host a group containing the default basemaps so that we can make use of the [`basemapsGroup`](https://developers.arcgis.com/javascript/3/jsapi/basemapgallery-amd.html#basemapgallery1-basemapsgroup) option.