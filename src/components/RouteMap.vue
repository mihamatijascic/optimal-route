<template>
  <l-map class="map" :zoom="zoom" :center="center" v-on:click="test">
    <l-tile-layer
      :url="baseLayer.url"
      :attribution="baseLayer.attribution"
    ></l-tile-layer>
    <l-wms-tile-layer
      :key="wmsLayer.name"
      :base-url="wmsLayer.url"
      :layers="wmsLayer.layers"
      :visible="wmsLayer.visible"
      :name="wmsLayer.name"
      :attribution="wmsLayer.attribution"
      :transparent="true"
      format="image/png"
      layer-type="base"
    >
    </l-wms-tile-layer>
    <l-geo-json
      :key="gjRestareasName"
      :geojson="gjRestareas"
      @click="test"
    ></l-geo-json>
    <l-geo-json :key="gjRestareasName" :geojson="gjRoute"></l-geo-json>
  </l-map>
</template>

<script>
import { LMap, LTileLayer, LWMSTileLayer, LGeoJson } from "vue2-leaflet";

export default {
  name: "RouteMap",
  props: {
    route: {
      type: String,
      required: false,
      default: "cite:highways_croatia",
    },
  },
  data() {
    return {
      totalRouteLength: 0,
      gjRoute: null,
      geojson_temp:
        "http://localhost:8080/geoserver/wfs?service=wfs&version=2.0.0&request=GetFeature&outputFormat=application/json&typeNames=",
      gjRestareas: null,
      gjRestareasName: "name",
      zoom: 7,
      center: [45.1, 16.25],
      baseLayer: {
        url: "https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",
        attribution:
          '&copy; <a target="_blank" href="http://osm.org/copyright">OpenStreetMap</a> contributors',
      },
      wmsLayer: {
        url: "http://localhost:8080/geoserver/wms",
        name: "cite:highways_croatia",
        visible: false,
        format: "image/png",
        layers: "cite:highways_croatia",
        transparent: true,
        attribution: "",
      },
    };
  },
  components: {
    LMap,
    LTileLayer,
    "l-wms-tile-layer": LWMSTileLayer,
    LGeoJson,
  },

  methods: {
    test: function (event) {
      this.$root.$emit(
        "send_selected_area",
        event["propagatedFrom"]["feature"]["properties"]
      );
    },
    sendRestAreas: function () {
      console.log("send rest areas");
      this.$root.$emit("send_rest_areas", this.gjRestareas, this.route["name"]);
    },
    getDistanceFromLatLonInKm: function (lat1, lon1, lat2, lon2) {
      var R = 6371; // Radius of the earth in km
      var dLat = this.deg2rad(lat2 - lat1); // deg2rad below
      var dLon = this.deg2rad(lon2 - lon1);
      var a =
        Math.sin(dLat / 2) * Math.sin(dLat / 2) +
        Math.cos(this.deg2rad(lat1)) *
          Math.cos(this.deg2rad(lat2)) *
          Math.sin(dLon / 2) *
          Math.sin(dLon / 2);
      var c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
      var d = R * c; // Distance in km
      return d;
    },
    calculateTotalLength: function() {
      var totalLength = 0;
      var coordinatesArray = this.gjRoute.features[0].geometry.coordinates[0];
      for (var i = 0; i < coordinatesArray.length - 1; i++) {
        var lon1 = coordinatesArray[i][0];
        var lat1 = coordinatesArray[i][1];
        var lon2 = coordinatesArray[i + 1][0];
        var lat2 = coordinatesArray[i + 1][1];

        totalLength += this.getDistanceFromLatLonInKm(lat1, lon1, lat2, lon2);
      }
      this.totalRouteLength = totalLength;
      console.log("total distance: " + totalLength.toFixed(3));
    },
    deg2rad(deg) {
      return deg * (Math.PI / 180);
    },
    async fetchWfsRestareas(newRoute){
      this.gjRestareasName = newRoute["value"] + "-restareas";
      var link = this.geojson_temp + this.gjRestareasName;
      const response = await fetch(link);
      var fullGeojson = await response.json();
      this.gjRestareas = fullGeojson["features"];
    },
    async fetchWfsShorthestPath(newRoute){
      var wfsPath = this.geojson_temp + newRoute["value"];
      var path = await fetch(wfsPath);
      this.gjRoute = await path.json();
    }
  },
  watch: {
    route: async function (newRoute) {
      this.wmsLayer.layers = newRoute["value"];
      this.wmsLayer.name = newRoute["value"];

      await this.fetchWfsShorthestPath(newRoute);

      console.log("shorthest path:");
      console.log(this.gjRoute);
      this.calculateTotalLength();

      await this.fetchWfsRestareas(newRoute);

      this.sendRestAreas();
    },
  },
};
</script>

<style scoped>
.map {
  height: 90vh;
}
</style>
