<template>
  <div class="container mt-3 mt-sm-5" id="app">
    <div class="row">
      <div class="col-md-8">
        <div class="map" id="map"></div>
      </div>
      <div class="col-md-4">
        <label>Encoded Polyline</label>
        <textarea v-model="encodedPolyline" placeholder="paste polyline"></textarea>
        <label>Decoded Polyline</label>
        <textarea v-model="jsonCoordinates"></textarea>
      </div>
    </div>
  </div>
</template>

<script>
import L from "leaflet";
import "bootstrap/dist/css/bootstrap.css";
import "bootstrap-vue/dist/bootstrap-vue.css";
import "leaflet/dist/leaflet.css";
var polyline = require('@mapbox/polyline');

export default {
  name: "app",
  data: function() {
    return {
      map: null,
      tileLayer: null,
      encodedPolyline: "",
      jsonCoordinates: "[]",
      coordinates: [],
      polyline: null
    };
  },
  watch: {
    encodedPolyline: function (val) {
      this.coordinates = polyline.decode(val);
    },
    jsonCoordinates: function (val) {
      this.coordinates = JSON.parse(val).map(function (latLng) {
        return [latLng.lat, latLng.lng];
      });
    },
    coordinates: function (val) {
      if (this.polyline == null) {
        this.polyline = L.polyline(val).addTo(this.map);
      } else {
        this.polyline.setLatLngs(val);
      }
      this.map.fitBounds(this.polyline.getBounds());
      this.encodedPolyline = polyline.encode(this.coordinates);
      this.jsonCoordinates = JSON.stringify(this.coordinates.map(function (coord) {
        return {lat: coord[0], lng: coord[1]};
      }), null, 4);
    }
  },
  mounted() {
    this.initMap();
  },
  methods: {
    initMap() {
      this.map = L.map("map").setView([40.1304, -75.5149], 12);
      this.tileLayer = L.tileLayer(
        "https://cartodb-basemaps-{s}.global.ssl.fastly.net/rastertiles/voyager/{z}/{x}/{y}.png",
        {
          maxZoom: 18,
          attribution:
            '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>, &copy; <a href="https://carto.com/attribution">CARTO</a>'
        }
      );
      this.tileLayer.addTo(this.map);
    }
  }
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.map {
  height: 600px;
}
</style>
