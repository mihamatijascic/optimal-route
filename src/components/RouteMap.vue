<template>
  <l-map class="map" :zoom="zoom" :center="center">
    <l-tile-layer
      :url="baseLayer.url"
      :attribution="baseLayer.attribution"
    ></l-tile-layer>
    <l-marker :lat-lng="markerLatLng" :icon="icons.restAreaIcon"> </l-marker>
    <l-geo-json :key="gjRestareasName" :geojson="gjRoute"></l-geo-json>
    <!-- item.icon.options.visible -->
    <l-marker
      v-for="(item,index) in latLngRestAreas"
      :key="'marker-'+index"
      :lat-lng="item.coordinates"
      :visible="showRestAreas"
      :icon="item.icon"
      @l-add="$event.target.openPopup()"
    >
    <l-popup :content="item.name"> </l-popup>
    </l-marker>
    <!-- <l-geo-json
      :key="gjRestareasName"
      :geojson="gjRestareas"
      @click="onMapClick"
      :visible="showRestAreas"
    ></l-geo-json> -->
    <!-- <l-geo-json :key="closestStation.id" :geojson="closestStation.stations" ></l-geo-json> -->
    <l-marker
      v-for="(item,index) in this.closestStation.stations"
      :key="'optimal-'+index"
      :lat-lng="item.coordinates"
      :icon="item.icon"
      @l-add="$event.target.openPopup()"
    >
    <l-popup :content="'Charging time: ' + item.chargingTime  + '<br>Station name: ' + item.name"></l-popup>
    </l-marker>
  </l-map>
</template>

<script>
import { LMap, LTileLayer, LGeoJson, LMarker, LPopup} from "vue2-leaflet";
import L from "leaflet";

export default {
  name: "RouteMap",
  props: {
    route: {
      type: Object,
      required: false,
      default: null,
    },
    optimize: {
      type: Boolean,
      required: false,
      default: false,
    },
  },
  components: {
    LMap,
    LTileLayer,
    LGeoJson,
    LMarker,
    LPopup
  },
  data() {
    return {
      closestStation: { id: 0, stations: null },
      showRestAreas: true,
      markerLatLng: [47.31322, -1.319482],
      totalRouteLength: 0,
      gjRoute: null,
      geojson_temp:
        "http://localhost:8080/geoserver/wfs?service=wfs&version=2.0.0&request=GetFeature&outputFormat=application/json&typeNames=",
      gjRestareas: null,
      latLngRestAreas: null,
      gjRestareasName: "name",
      zoom: 7,
      center: [45.1, 16.25],
      icons: {
        restAreaIcon: L.icon({
          iconUrl: require("@/assets/icons8-coffee-50.png"),
          iconSize: [32, 37],
          iconAnchor: [16, 37],
          visible: true,
        }),
        fuelIcon: L.icon({
          iconUrl: require("@/assets/icons8-fuel-64.png"),
          iconSize: [32, 37],
          iconAnchor: [16, 37],
          visible: true,
        }),
        parkingIcon: L.icon({
          iconUrl: require("@/assets/icons8-parking-48.png"),
          iconSize: [32, 37],
          iconAnchor: [16, 37],
          visible: true,
        }),
      },
      baseLayer: {
        url: "https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",
        attribution:
          '&copy; <a target="_blank" href="http://osm.org/copyright">OpenStreetMap</a> contributors',
      },
    };
  },
  mounted() {
    this.$root.$on("route_selected", (selectedRoute)=>{
      console.log("recieved selected route!!");
      console.log(selectedRoute);
      this.route = selectedRoute;
    });
    this.$root.$on(
      "send_carRemainingDistance",
      (remaningDistance, maxRange, battery, minPower, maxPower) => {
        console.log("remDistance: " + remaningDistance + ", maxRange: " + maxRange + ", battery: " + battery + ", minPower: " + minPower  + ", maxPower: " + maxPower);
        var totalLength = 0;
        var linePoints = this.gjRoute.features[0].geometry.coordinates[0];
        var lastClosestPoint = this.calcClosestPoint(
          linePoints[0][0],
          linePoints[0][1]
        );
        var optimalChargingStationPositions = [];
        var totalLengthOfLastClosestStation = 0;

        for (var i = 0; i < linePoints.length - 1; i++) {
          var lon1 = linePoints[i][0];
          var lat1 = linePoints[i][1];
          var lon2 = linePoints[i + 1][0];
          var lat2 = linePoints[i + 1][1];

          var step = this.getDistanceFromLatLonInKm(lat1, lon1, lat2, lon2);
          totalLengthOfLastClosestStation += step;
          totalLength += step;

          if (totalLength > remaningDistance) {
            var stationMarker = this.createJsonMarker(lastClosestPoint);
            var batteryToFillPerc = (maxRange - totalLengthOfLastClosestStation)/maxRange;
            console.log("battery: " + battery);
            console.log("battery to fill: " + batteryToFillPerc);
            console.log("battery in kWh: " + (battery * batteryToFillPerc));
            var minTime = (battery * batteryToFillPerc) / maxPower;
            var maxTime = (battery * batteryToFillPerc) / minPower;
            stationMarker.chargingTime = minTime.toFixed(1) + " - " + maxTime.toFixed(1) + " hours";
            
            optimalChargingStationPositions.push(stationMarker);
            remaningDistance += maxRange - totalLengthOfLastClosestStation;
          }

          var closestPointForCurrentPoint = this.calcClosestPoint(lon2, lat2);
          if (closestPointForCurrentPoint == null) continue;

          lastClosestPoint = closestPointForCurrentPoint;
          totalLengthOfLastClosestStation = 0;
        }

        console.log("after latLng optimal latlng:");
        console.log(optimalChargingStationPositions);
        this.totalRouteLength = totalLength;
        this.closestStation = {
          id: this.closestStation.id + 1,
          stations: optimalChargingStationPositions,
        };
        console.log("ater latLng optimal latlng:");
        console.log(this.closestStation.stations);
        this.optimize = true;
      }
    );
  },
  methods: {
    calcClosestPoint(lon, lat) {
      var minDistance = 100000000;
      var minPoint = null;
      const radius = 2;

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
    onMapClick: function (event) {
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
    deg2rad(deg) {
      return deg * (Math.PI / 180);
    },
    conditionalIcon(element){
      if(element.properties.amenity == "parking"){
        return this.icons.parkingIcon;
      }else if(element.properties.highway == "passing_place" || element.properties.highway == "rest_area"){
        return this.icons.restAreaIcon;
      }else{
        return this.icons.fuelIcon;
      }
    },
    createJsonMarker(element){
      var lon = element.geometry.coordinates[0];
      var lat = element.geometry.coordinates[1];
      var markerJson = {
        name: element.properties.name,
        properties: element.properties,
        coordinates: L.latLng(lat, lon),
        icon: this.conditionalIcon(element)
      }
      return markerJson;
    },
    createLngLatArray(gjArray){
      this.latLngRestAreas = []
      var latLngArray = []
      console.log("before new loc");

      gjArray.forEach(element => {
        var latLng = this.createJsonMarker(element);
        latLngArray.push(latLng);
      });

      console.log(this.latLngRestAreas);
      return latLngArray;
    },
    async fetchWfsRestareas(newRoute) {
      this.gjRestareasName = newRoute["value"] + "-restareas";
      var link = this.geojson_temp + this.gjRestareasName;
      const response = await fetch(link);
      var fullGeojson = await response.json();
      this.gjRestareas = fullGeojson["features"];

      this.latLngRestAreas = this.createLngLatArray(this.gjRestareas);
      // this.createLngLatRestAreas();
    },
    async fetchWfsShorthestPath(newRoute) {
      var wfsPath = this.geojson_temp + newRoute["value"];
      var path = await fetch(wfsPath);
      this.gjRoute = await path.json();
    },
  },
  watch: {
    route: async function (newRoute) {
      await this.fetchWfsShorthestPath(newRoute);

      console.log("shorthest path:");
      console.log(this.gjRoute);

      await this.fetchWfsRestareas(newRoute);
      console.log("rest areas:");
      console.log(this.gjRestareas);

      this.sendRestAreas();
      this.optimize = false;
    },
    optimize: function (newOptimize) {
      if (newOptimize == true) {
        this.showRestAreas = false;
      } else {
        this.showRestAreas = true;
        this.closestStation.stations = [];
        this.closestStation.id += 1;
      }
    },
  },
};
</script>

<style scoped>
.map {
  height: 90vh;
}
</style>
