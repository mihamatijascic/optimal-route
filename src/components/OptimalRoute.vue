<template>
  <div class="container">
    <div class="row">
      <div class="col-12">
        <h1 class="text-center text-info">Route list</h1>
      </div>
    </div>
    <div class="row">
      <div class="col-6">
        <select @change="onChange" v-model="selected">
          <option
            v-bind:key="index"
            v-for="(route, index) in routes"
            v-bind:value="route.value"
          >
            {{ route.name }}
          </option>
        </select>
        <!--<span>{{selected}}</span>-->
      </div>
      <div class="col-6">
        <RouteMap :route="selected" />
      </div>
    </div>
  </div>
</template>

<script>
import RouteMap from "./RouteMap.vue";

export default {
  components: { RouteMap },
  name: "OptimalRoute",
  data: function () {
    return {
      selected: "cite:highways_croatia",
      routes: [
        { name: "Croatia highways", value: "cite:highways_croatia" },
        { name: "Zagreb-Split", value: "cite:Zagreb-Split" },
        { name: "Zagreb-Rijeka", value: "cite:Zagreb-Rijeka" },
        { name: "Zagreb-Osijek", value: "cite:Zagreb-Osijek" },
        { name: "Split-Rijeka", value: "cite:Split-Rijeka" },
        { name: "Split-Osijek", value: "cite:Split-Osijek" },
        { name: "Rijeka-Osijek", value: "cite:Rijeka-Osijek" },
      ],
    };
  },
  methods: {
    onChange() {
      console.log(this.selected);
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped></style>
