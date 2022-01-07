<template>
  <div class="container my-2">
    <h5>Rest areas for {{ routeName }}</h5>
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
</template>

<script>
export default {
  name: "RestAreasTable",
  data() {
    return {
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
  },
};
</script>

<style scoped>
.container {
  max-height: 300px;
  overflow-y: scroll;
}
</style>
