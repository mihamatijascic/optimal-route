<template>
  <div class="container my-2" v-if="restAreas">
    <h5>Rest areas for: {{ routeName }}</h5>
    <div id="table" class="my-2">
      <table class="table table-striped table-bordered sticky-header">
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
          </tr>
        </tbody>
      </table>
    </div>
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
  max-height: 15rem;
  overflow-y: scroll;
}
.table thead {
  position: sticky;
  text-align: center;
}
.table thead tr th{
  top: 0;
}
.table tr {
  text-align: center;
}
</style>
