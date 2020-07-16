<template>
  <div class="hello">
    <div style="width:500px; margin:0 auto; justify-content: center">
    geoJSON Picker | &copy;  clime.ai 
    </div>

    <l-map
    ref="map"
    style="height:500px; width: 50%; margin: auto;"
    :zoom="zoom"
    :center="center"
    :options="mapOptions"
    @update:center="centerUpdated"
    >

    <v-geo-search
    :options="geosearchOptions"
    ></v-geo-search>

     <l-tile-layer
      v-for="tileProvider in tileProviders"
      :key="tileProvider.name"
      :name="tileProvider.name"
      :visible="tileProvider.visible"
      :url="tileProvider.url"
      :attribution="tileProvider.attribution"
      layer-type="base"
      />

    <l-control-attribution position="bottomright" prefix="clime.ai"></l-control-attribution>
    <l-control-layers position="bottomright"  ></l-control-layers>


    </l-map>

      <div> 

        Current center: lat {{lat}} lon {{ lon }}

      </div>

        <button @click="printGeoJson" style="margin-top: 20px;">
          print geoJSON
        </button>

      <div id="geoJSON" style="margin: 15px; border:1px solid black; padding: 20px;">

      </div>

      <h1> Or paste in geoJSON and let it display on the map </h1>

      <input type="text" id="jsonString" value="">

      <button @click="GeoJSONtoMap" style="margin-top: 20px;">
          Submit
      </button>



  </div>
</template>

<script>
import {LMap, LTileLayer, LControlAttribution, LControlLayers } from 'vue2-leaflet';
import { OpenStreetMapProvider } from 'leaflet-geosearch';
import VGeoSearch from './GeoSearch.vue';
import 'leaflet-draw';


export default {  
  name: 'HelloWorld',
  components: {
    LMap,
    LControlAttribution,
    LTileLayer,
    LControlLayers,
    VGeoSearch
  },
  props: {
  },

  data: () => ({
      //map data
      center: [50.045730621731266, 8.22232246398926],
      zoomanimation: true,
      zoom: 15,
      lat: 50.045730621731266,
      lon: 8.22232246398926,
      map_initialized: false,
      map: Object,
      editableLayers: Object,
      drawControl: Object,
      tileProviders: [
        {
          name: 'OpenStreetMap',
          visible: true,
          attribution:
            'Map: &copy; <a target="_blank" href="http://osm.org/copyright">OSM</a> contributors',
          url: 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',
        },
        {
          name: 'Satelitte',
          visible: false,
          attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, <a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
          url: 'https://api.mapbox.com/styles/v1/mapbox/satellite-streets-v11/tiles/256/{z}/{x}/{y}?access_token=pk.eyJ1IjoibWFpbnZvaWQiLCJhIjoiY2pjdnk4OHl5MTdqNDJ4bnVub2FuNDlhOCJ9.ij3l4GJ_KBOHmDKDYXL__w'
        },
        {
          name: 'OpenTopoMap',
          visible: false,
          url: 'https://{s}.tile.opentopomap.org/{z}/{x}/{y}.png',
          attribution:
            'Map data: &copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>, <a href="http://viewfinderpanoramas.org">SRTM</a> | Map style: &copy; <a href="https://opentopomap.org">OpenTopoMap</a> (<a href="https://creativecommons.org/licenses/by-sa/3.0/">CC-BY-SA</a>)',
        }
      ],
      mapOptions: {attributionControl: false},
      geosearchOptions: { // Important part Here
        provider: new OpenStreetMapProvider(),
        autoComplete: true,
        autoClose: true, // optional: true|false  - default false
        showMarker: false,
        notFoundMessage: "We could not find this adress.",
        searchLabel: "Type in a place...",
        zoomLevel: 9,
        style:'bar',
        showPopup: false,
        retainZoomLevel: false,
        animateZoom: true,
        keepResult: true,
        }
  }),

  

  mounted: function() {
              this.$nextTick(() => {
              const map = this.$refs.map.mapObject;
              this.map = map;
              map.addControl(new window.L.Control.Fullscreen());
              this.map_initialized = true;

              this.editableLayers = new window.L.FeatureGroup().addTo(map);

              this.drawControl = new window.L.Control.Draw({
                position: 'topright',
                draw: {
                  polyline: false,
                  //{
                  //  allowIntersection: false,
                  //  showArea: true
                  //},
                  polygon: true,
                  rectangle: true,
                  circle: true,
                  marker: true
                },
                edit: {
                  featureGroup: this.editableLayers,
                  polyline: false,
                  polygon: true,
                  rectangle: true,
                  circle: true,
                  edit: true
                }
              });


          
              
              map.addControl(this.drawControl);

              map.on(window.L.Draw.Event.CREATED, (e) => {
                // const type = e.layerType;
                const layer = e.layer;
                // Do whatever else you need to. (save to db, add to map etc)
                this.editableLayers.addLayer(layer); 
                this.printGeoJson();           
              });
            });
  },

  updated: function() {

      this.lat = this.center.lat;
      this.lon = this.center.lng;

  },

  methods: {

    printGeoJson: function() {

      let geojson = this.editableLayers.toGeoJSON()
      document.getElementById("geoJSON").innerHTML = JSON.stringify(geojson)

    },
    centerUpdated (center){
          this.center = center;
      },

    GeoJSONtoMap() {
    var geojson = JSON.parse((document.getElementById("jsonString").value));
    window.L.geoJSON(geojson, {onEachFeature: this.layerToMap})
    },

    layerToMap(feature,layer){
      this.editableLayers.addLayer(layer);
    }


  }

}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
ul {
  list-style-type: none;
}
a {
  color: #42b983;
}
</style>
