<template>
  <div class="container">
    <h5>Rest areas for: {{ routeName }}</h5>
    <table id="table" class="table table-striped table-bordered sticky-header">
      <thead>
        <tr>
          <th>Index</th>
          <th>Name</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(rest_area, index) in restAreas" :key="index">
          <td>{{ index + 1 }}</td>
          <td>{{ rest_area.properties.name }}</td>
          <!-- <td>{{ rest_area.name }}</td> -->
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
      selectedArea: null,
      restAreas: null,
      routeName: null,
    };
  },
  mounted() {
    this.$root.$on("send_rest_areas", (rest_areas, route_name) => {
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
}
.table th {
  text-align: center;
}
.table thead tr th {
  position: sticky;
  top: 0;
}
.table tr {
  text-align: center;
}
</style>
