<template>
<v-content>
  <div  >

    <img :src="require(`@/assets/logo.png`)" style="width:10%; margin-top:0px; padding-top:0px;" />

    <div style="width:400px; margin:10px auto; justify-content: center">
    geoJSON Picker | &copy;  clime.ai 
    </div>

    <v-row    
    class="black--text">

    <l-map
    ref="map"
    style="height:800px; width: 50%; margin: auto; margin-left: 10px;"
    :zoom="zoom"
    :center="center"
    :options="mapOptions"
    @update:center="centerUpdated"
    class="elevation-10"
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

      <v-virtual-scroll
        :items="geoJSONarray"
        height="800"
        item-height="800"
      >

     <vue-json-pretty
      v-if="geoJSONbool"
      :data="geoJSON"
      @click="doNothing"
      :showLine="false"
      :highlightMouseoverNode="true"
      :showLength="true"
      id="geoJSONpretty"
      >
      </vue-json-pretty>

      </v-virtual-scroll>

    </v-row>

    <v-row class="text-center" align="center" justify="center" style="margin:auto; justify-content: center; margin-bottom: 30px; margin-top: 0px;" >

      <v-btn
      v-clipboard:copy="geoJSONstring"
      v-clipboard:success="onCopy"
      v-clipboard:error="onError"
      class="primary white--text ma-5 elevation-10"
      >
      Copy geoJSON to Clipboard
      </v-btn>

    </v-row>

    <v-row class="text-center" align="center" justify="center" style="margin:auto; justify-content: center; margin-bottom: 30px; margin-top: 0px;" >

    <v-card
    v-if="lat && lon"
    elevation="12"
    width="600px"
    class="pa-8"
    >

       <div v-if="featureAmount != undefined">
          Selected features: {{featureAmount}}
      </div>


      <div v-if="lat && lon" style="margin: 20px;"> 
        Current center: lat {{lat}} lon {{lon}}
      </div>


    </v-card>

    </v-row>

      <h3 style="padding-top:30px;"> Paste in a geoJSON string and display it on the map </h3>

      <v-row style="padding-top: 20px; padding-left: 50px; padding-right: 50px;">

        
      <v-text-field

      label="Paste your geoJSON"
      v-model="inputJSONstring"

      ></v-text-field>

      </v-row>

      <v-row class="text-center" align="center" justify="center" style="margin:auto; justify-content: center; margin-bottom: 30px; margin-top: 0px;" >

      <v-btn @click="GeoJSONtoMap" 
      style="margin-top: 0px;" 
      class="primary white--text ma-5 elevation-10">
          Submit
      </v-btn>

      </v-row>

  </div>

</v-content>

</template>

<script>
import {LMap, LTileLayer, LControlAttribution, LControlLayers } from 'vue2-leaflet';
import { OpenStreetMapProvider } from 'leaflet-geosearch';
import VGeoSearch from './GeoSearch.vue';
import 'leaflet-draw';
import VueJsonPretty from 'vue-json-pretty';


export default {  
  name: 'HelloWorld',
  components: {
    LMap,
    LControlAttribution,
    LTileLayer,
    LControlLayers,
    VGeoSearch,
    VueJsonPretty
  },
  props: {
  },

  data: () => ({
      //map data
      center: [50.045730621731266, 8.22232246398926],
      zoomanimation: true,
      zoom: 15,
      lat: "",
      lon: "",
      map_initialized: false,
      map: Object,
      editableLayers: Object,
      drawControl: Object,
      geoJSON: {},
      geoJSONstring: "",
      inputJSONstring: undefined,
      featureAmount: undefined,
      maxFeatures: 5,
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
                  circle: false,
                  marker: true,
                  circlemarker: false
                },
                edit: {
                  featureGroup: this.editableLayers,
                  polyline: false,
                  polygon: true,
                  rectangle: true,
                  circle: false,
                  circlemarker: false,
                  edit: true
                }
              });


          
              
              map.addControl(this.drawControl);

              map.on(window.L.Draw.Event.CREATED, (e) => {

                if (this.featureAmount == undefined || this.featureAmount < this.maxFeatures) {
                // const type = e.layerType;
                const layer = e.layer;
                // Do whatever else you need to. (save to db, add to map etc)
                this.editableLayers.addLayer(layer);                
                this.featureAmount = Object.keys(this.editableLayers._layers).length; 
                this.printGeoJson(); 
                }
                else {
                  window.alert("This is not allowed!")
                }
              });

              map.on(window.L.Draw.Event.EDITED, () => {
                this.geoJSON = this.editableLayers.toGeoJSON();

              });

              map.on(window.L.Draw.Event.DELETED, () => {
                this.geoJSON = this.editableLayers.toGeoJSON();

              });

              })
              
  },

  updated: function() {

      this.lat = this.center.lat;
      this.lon = this.center.lng;

  },

  methods: {

    printGeoJson: function() {
      let geojson = this.editableLayers.toGeoJSON()
      let geoJSONstring = JSON.stringify(geojson);
      this.geoJSONstring = geoJSONstring;
      this.geoJSON = geojson;
    },
    centerUpdated (center){
          this.center = center;
      },

    GeoJSONtoMap() {

      if (this.inputJSONstring != undefined && this.inputJSONstring != ""){
        try {
          var geoJSON = JSON.parse(this.inputJSONstring);
          window.L.geoJSON(geoJSON, {onEachFeature: this.layerToMap})
          this.geoJSON = this.editableLayers.toGeoJSON();
          
        } catch (error) {
          window.alert("The geoJSON is not valid.")
          
        }
          

      }
      else  { window.alert("You need to type in a geoJSON to submit.")}


    
    },

    layerToMap(feature,layer){
      this.editableLayers.addLayer(layer);
    },

    doNothing() {

    },

    onCopy: function (e) {
      alert('You just copied: ' + e.text)
    },
    onError: function () {
      alert('Failed to copy texts')
    }


  },
  computed: {
    geoJSONbool: function() {
      if ( (Object.keys(this.geoJSON).length) == 0) {
        return false
      }
      else {
        return true
      }

    },
    geoJSONarray: function() {
      return [this.geoJSON]
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
