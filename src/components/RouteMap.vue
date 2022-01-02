<template>
  <div class="row map">
    <l-map :zoom="zoom" :center="center">
      <l-tile-layer :url="baseLayer.url" :attribution="baseLayer.attribution"></l-tile-layer>
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
    </l-map>
  </div>
</template>

<script>
import L from "leaflet";
import { LMap, LTileLayer, LWMSTileLayer } from "vue2-leaflet";

export default {
  name: "RouteMap",
  props: {
      route: {
          type: String,
          required: false,
          default: "cite:highways_croatia"
      }
  },
  data() {
    return {
      zoom: 7,
      center: [45.1, 15.2],
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
        attribution:
          '',
      },
    };
  },
  components: {
    LMap,
    LTileLayer,
    "l-wms-tile-layer": LWMSTileLayer,
  },

  methods: {
    latLng: function (lat, lng) {
      return L.latLng(lat, lng);
    },
  },
  watch:{
      route: function(newRoute){
          this.wmsLayer.layers = newRoute;
          this.wmsLayer.name = newRoute;
      }
  }
};
</script>

<style scoped>
.map {
  height: 95vh;
}
</style>
