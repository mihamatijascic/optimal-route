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
    <l-geo-json :key="geojsonName" :geojson="geojson"></l-geo-json>
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
      geojson_temp: "http://localhost:8080/geoserver/wfs?service=wfs&version=2.0.0&request=GetFeature&outputFormat=application/json&typeNames=",
      geojson: null,
      geojsonName: "name",
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
        visible: true,
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
    test: function(event){
      console.log(event);
    },
  },
  watch: {
    route: async function (newRoute) {
      this.wmsLayer.layers = newRoute["value"];
      this.wmsLayer.name = newRoute["value"];
      
      this.geojsonName = newRoute["value"] + "-restareas"
      console.log(this.geojsonName)
      var link = this.geojson_temp + this.geojsonName;
      const response = await fetch(link);
      this.geojson = await response.json();
    },
  },
  // async created() {
  //   // var link = this.geojson_temp + newRoute["value"] + "-restareas";
  //   const response = await fetch(
  //     "http://localhost:8080/geoserver/wfs?service=wfs&version=2.0.0&request=GetFeature&outputFormat=application/json&typeNames=cite:Rijeka-Osijek-restareas"
  //   );
  //   this.geojson = await response.json();
  //   console.log(this.geojson);
  // },
};
</script>

<style scoped>
.map {
  height: 90vh;
}
</style>
