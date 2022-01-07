<template>
  <div class="container my-2">
    <h5>Rest areas for: {{ routeName }}</h5>
    <div id="table" class="my-2">
      <table class="table table-striped table-bordered">
        <thead>
          <tr>
            <th>Index</th>
            <th>Name</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(rest_area, index) in restAreas" :key="index">
            <td>{{ index + 1 }}.</td>
            <td>{{ rest_area.properties.name }}</td>
          </tr>
        </tbody>
      </table>
    </div>
    <h5>Selected rest area: {{this.selectedArea}}</h5>
  </div>
</template>

<script>
export default {
  name: "RestAreasTable",
  data() {
    return {
      selectedArea: null,
      restAreas: null,
      routeName: null,
    };
  },
  mounted() {
    this.$root.$on("send_rest_areas", (rest_areas, route_name) => {
      console.log("rest areas recieved!");
      
      this.restAreas = rest_areas;
      this.routeName = route_name;
    });
    this.$root.$on("send_selected_area", (area) => {
      this.selectedArea = area["name"];
    });
  },
};
</script>

<style scoped>
#table {
  max-height: 300px;
  overflow-y: scroll;
}
</style>
