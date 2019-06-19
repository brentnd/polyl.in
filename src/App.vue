<template>
  <div class="container-fluid h-100" id="app">
    <div class="row h-100">
      <div class="col-md-8">
        <div class="map" id="map"></div>
      </div>
      <div class="col-md-4">
        <div class="form-group">
          <label>Encoded Polyline</label>
          <textarea class="form-control" v-model="encodedPolyline" placeholder="oqr~FtmzuOJxjAiN~B"></textarea>
          <a href="#" v-on:click="coordinates = []" v-show="coordinates.length > 0">Clear</a>
        </div>
        <div class="form-group">
          <label>Decoded Coordinates</label>
          <table class="table">
            <thead>
              <tr>
                <th scope="col">#</th>
                <th scope="col">Latitude</th>
                <th scope="col">Longitude</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(coordinate, index) in coordinates" v-bind:key="index">
                <th scope="row">{{ index + 1 }}</th>
                <td>
                  <input
                    class="form-control input-lg"
                    type="number"
                    step="any"
                    v-model="coordinate[0]"
                  >
                </td>
                <td>
                  <input
                    class="form-control input-lg"
                    type="number"
                    step="any"
                    v-model="coordinate[1]"
                  >
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import L from "leaflet";
import "bootstrap/dist/css/bootstrap.css";
import "bootstrap-vue/dist/bootstrap-vue.css";
import "leaflet/dist/leaflet.css";
var polyline = require("@mapbox/polyline");

export default {
  name: "app",
  data: function() {
    return {
      map: null,
      tileLayer: null,
      encodedPolyline: "",
      coordinates: [],
      polyline: null
    };
  },
  watch: {
    encodedPolyline: function(val) {
      this.coordinates = polyline.decode(val);
    },
    coordinates: function(val) {
      if (val.length > 0) {
        if (this.polyline == null) {
          this.polyline = L.polyline(val).addTo(this.map);
        } else {
          this.polyline.setLatLngs(val);
        }
        this.map.fitBounds(this.polyline.getBounds());
        this.encodedPolyline = polyline.encode(this.coordinates);
      } else {
        this.defaultMap();
        this.polyline = null;
        this.encodedPolyline = "";
      }
    }
  },
  mounted() {
    this.initMap();
  },
  methods: {
    initMap() {
      this.map = L.map("map");
      this.defaultMap();
      this.tileLayer = L.tileLayer(
        "https://cartodb-basemaps-{s}.global.ssl.fastly.net/rastertiles/voyager/{z}/{x}/{y}.png",
        {
          maxZoom: 18,
          attribution:
            '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>, &copy; <a href="https://carto.com/attribution">CARTO</a>'
        }
      );
      this.tileLayer.addTo(this.map);
    },
    defaultMap() {
      this.map.setView([40.1304, -75.5149], 12);
    }
  }
};
</script>

<style>
html,
body {
  height: 100%;
}
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  height: 100%;
}
.map {
  height: 100%;
}
</style>
