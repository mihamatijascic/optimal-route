<template>
  <div class="no-scroll grid-container m-5">
    <div class="map">
      <RouteMap/>
    </div>
    <div class="relations dropdown">
      <configuration-form/>
      <rest-areas-table/>
    </div>
  </div>
</template>

<script>
import RouteMap from "./RouteMap.vue";
import RestAreasTable from "./RestAreasTable.vue";
import ConfigurationForm from "./ConfigurationForm.vue";

export default {
  components: { RouteMap, RestAreasTable, ConfigurationForm },
  name: "OptimalRoute",
  mounted() {
    
  },
  data() {
    return {
      
    };
  },
  methods:{
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.no-scroll{
  overflow-y: none;
}
.relations {
  height: 90vh;
  text-align: left;
  z-index: 1200;
}
.grid-container {
  display: grid;
  justify-content: start;
  grid-template-columns: 0.65fr 0.35fr;
  grid-column-gap: 50px;
}
</style>

<!-- 
{ name: "Croatia highways", value: "cite:highways_croatia" },
{ name: "Zagreb-Split", value: "cite:Zagreb-Split" },
{ name: "Zagreb-Rijeka", value: "cite:Zagreb-Rijeka" },
{ name: "Zagreb-Osijek", value: "cite:Zagreb-Osijek" },
{ name: "Split-Rijeka", value: "cite:Split-Rijeka" },
{ name: "Split-Osijek", value: "cite:Split-Osijek" },
{ name: "Rijeka-Osijek", value: "cite:Rijeka-Osijek" },


{ name: "Croatia highways", value: "cite:highways_croatia" },
{ name: "Zagreb-Split", value: "Zagreb-Split-fuel-stations" },
{ name: "Zagreb-Rijeka", value: "Zagreb-Rijeka-fuel-stations" },
{ name: "Zagreb-Osijek", value: "Zagreb-Osijek-fuel-stations" },
{ name: "Split-Rijeka", value: "Split-Rijeka-fuel-stations" },
{ name: "Split-Osijek", value: "Split-Osijek-fuel-stations" },
{ name: "Rijeka-Osijek", value: "Rijeka-Osijek-fuel-stations" },
-->
