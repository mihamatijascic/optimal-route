<template>
  <div class="container my-2">
    <div class="my-2">
      <h5>Electric vehicles:</h5>
      <multiselect
        v-model="selectedCar"
        :options="cars"
        searchable
        track-by="name"
        label="name"
        allow-empty
        placeholder="Select a car"
      ></multiselect>
    </div>
    <div id="slider" class="my-4">
      <h5 id="battery">Battery fill percentage:</h5>
      <vue-slider v-model="bateryStatus" :min="minSlider" :max="maxSlider">
      </vue-slider>
    </div>
  </div>
</template>

<script>
import VueSlider from "vue-slider-component";
import "vue-slider-component/theme/default.css";

export default {
  name: "ConfigurationForm",
  components: {
    VueSlider,
  },
  data() {
    return {
      bateryStatus: 10,
      minSlider: 0,
      maxSlider: 100,
      selectedCar: null,
      cars: null,
    };
  },
  mounted() {
    this.cars = this.getCars();
    this.selectedCar = this.cars[0];
  },
  methods: {
    getCars: function () {
      return [
        { name: "fiesta", batery: 100 },
        { name: "punto", batery: 0 },
        { name: "twingo", batery: 10000 },
      ];
    },
  },
};
</script>

<style scoped>
.row {
  text-align: left;
  margin: 5px;
  z-index: 1000;
}
#battery {
  text-align: left;
}
#slider {
  z-index: 1;
}
</style>
