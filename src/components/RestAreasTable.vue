<template>
  <div class="container mt-4">
    <table class="table table-striped table-bordered">
        <thead>
            <tr>
                <th>Index</th>
                <th>Name</th>
            </tr>
        </thead>
        <tbody>
            <tr v-for="(rest_area, index) in restAreas" :key="index">
                <td>{{index}}</td>
                <td>{{rest_area.properties.name}}</td>
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
    };
  },
  mounted() {
    this.$root.$on("send_rest_areas", (rest_areas) => {
      console.log("rest areas recieved!");
      this.restAreas = rest_areas;
    });
  },
};
</script>

<style scoped>

</style>
