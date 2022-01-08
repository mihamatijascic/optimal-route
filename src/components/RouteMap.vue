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
    <l-marker :lat-lng="markerLatLng"></l-marker>
    <l-geo-json
      :key="gjRestareasName"
      :geojson="gjRestareas"
      @click="test"
      :visible="showRestAreas"
    ></l-geo-json>
    <l-geo-json :key="closestPoint.id" :geojson="closestPoint.stations" />
    <l-geo-json :key="gjRestareasName" :geojson="gjRoute"></l-geo-json>
  </l-map>
</template>

<script>
import {
  LMap,
  LTileLayer,
  LWMSTileLayer,
  LGeoJson,
  LMarker,
} from "vue2-leaflet";

export default {
  name: "RouteMap",
  props: {
    route: {
      type: String,
      required: false,
      default: "cite:highways_croatia",
    },
  },
  components: {
    LMap,
    LTileLayer,
    "l-wms-tile-layer": LWMSTileLayer,
    LGeoJson,
    LMarker,
  },
  data() {
    return {
      closestPoint: { id: 0, stations: null },
      showRestAreas: true,
      markerLatLng: [47.31322, -1.319482],
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
  mounted() {
    this.$root.$on(
      "send_carRemainingDistance",
      (remaningDistance, maxRange) => {
        var totalLength = 0;
        var linePoints = this.gjRoute.features[0].geometry.coordinates[0];
        var lastClosestPoint = this.calcClosestPoint(linePoints[0][0], linePoints[0][1]);
        var optimalChargingStationPositions = [];

        for (var i = 0; i < linePoints.length - 1; i++) {
          var lon1 = linePoints[i][0];
          var lat1 = linePoints[i][1];
          var lon2 = linePoints[i + 1][0];
          var lat2 = linePoints[i + 1][1];

          totalLength += this.getDistanceFromLatLonInKm(lat1, lon1, lat2, lon2);

          if (totalLength > remaningDistance) {
            optimalChargingStationPositions.push(lastClosestPoint);
            remaningDistance += maxRange;
          }

          var closestPointForCurrentPoint = this.calcClosestPoint(lon2, lat2);
          if (closestPointForCurrentPoint == null) continue;

          lastClosestPoint = closestPointForCurrentPoint;
        }

        this.totalRouteLength = totalLength;
        this.closestPoint = {
          id: this.closestPoint.id + 1,
          stations: optimalChargingStationPositions,
        };
        console.log(optimalChargingStationPositions);
        this.showRestAreas = false;
      }
    );
  },
  methods: {
    calcClosestPoint(lon, lat) {
      var minDistance = 100000000;
      var minPoint = null;
      const radius = 5;

      for (var point of this.gjRestareas) {
        var pointLon = point.geometry.coordinates[0];
        var pointLat = point.geometry.coordinates[1];
        var distance = this.getDistanceFromLatLonInKm(
          lat,
          lon,
          pointLat,
          pointLon
        );

        if (distance < minDistance && distance < radius) {
          minDistance = distance;
          minPoint = point;
        }
      }
      return minPoint;
    },
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
    calculateTotalLength: function () {
      var totalLength = 0;
      var linePoints = this.gjRoute.features[0].geometry.coordinates[0];
      for (var i = 0; i < linePoints.length - 1; i++) {
        var lon1 = linePoints[i][0];
        var lat1 = linePoints[i][1];
        var lon2 = linePoints[i + 1][0];
        var lat2 = linePoints[i + 1][1];

        totalLength += this.getDistanceFromLatLonInKm(lat1, lon1, lat2, lon2);
      }
      this.totalRouteLength = totalLength;
      console.log("total distance: " + totalLength.toFixed(3));
    },
    deg2rad(deg) {
      return deg * (Math.PI / 180);
    },
    async fetchWfsRestareas(newRoute) {
      this.gjRestareasName = newRoute["value"] + "-restareas";
      var link = this.geojson_temp + this.gjRestareasName;
      const response = await fetch(link);
      var fullGeojson = await response.json();
      this.gjRestareas = fullGeojson["features"];
      this.showRestAreas = true;
      this.closestPoint.stations = [];
      this.closestPoint.id += 1;
    },
    async fetchWfsShorthestPath(newRoute) {
      var wfsPath = this.geojson_temp + newRoute["value"];
      var path = await fetch(wfsPath);
      this.gjRoute = await path.json();
    },
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
      console.log("rest areas:");
      console.log(this.gjRestareas);

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
