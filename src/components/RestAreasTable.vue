<template>
  <div class="container mt-3">
    <h3 class="label label-primary">Rest areas for {{ routeName }}</h3>
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
.container{
    max-height: 500px;
    overflow: auto;
}
</style>
