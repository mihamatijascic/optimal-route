<template>
  <div class="grid-container m-5">
    <div class="map">
      <RouteMap />
    </div>
    <div class="relations dropdown">
      <configuration-form />
      <rest-areas-table />
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
  mounted() {},
  data() {
    return {};
  },
  methods: {},
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.relations {
  height: 90vh;
  text-align: left;
}
.grid-container {
  display: grid;
  justify-content: start;
  grid-template-columns: 0.65fr 0.35fr;
  grid-column-gap: 50px;
}
</style>
