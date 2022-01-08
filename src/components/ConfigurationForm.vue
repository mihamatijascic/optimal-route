<template>
  <div class="container my-4">
    <div class="my-2">
      <h5>Electric vehicle model:</h5>
      <multiselect
        v-model="selectedCar"
        :options="cars"
        searchable
        track-by="name"
        :customLabel="customLabelCar"
        allow-empty
        placeholder="Select a car"
        id="electric"
      ></multiselect>
    </div>
    <div class="my-4">
      <h5>Charging station type:</h5>
      <multiselect
        v-model="selectedStation"
        :options="chargingStations"
        searchable
        track-by="name"
        :customLabel="customLabelStation"
        allow-empty
        placeholder="Select charging station type"
        id="electric"
      ></multiselect>
    </div>
    <div class="form-floating my-4">
      <h5>Battery fill percentage: {{ batteryStatus }}%</h5>
      <vue-slider
        id="slider"
        v-model="batteryStatus"
        :min="minSlider"
        :max="maxSlider"
        :contained="sliderContained"
        v-on:change="calcChargingTime"
      >
      </vue-slider>
    </div>
    <div class="grid-container my-4">
      <div class="row">
        <h2 class="col btn btn-secondary btn-lg" style="vertical-align: middle;">
          Charging time:
        </h2>
        <h4 class="col" style="text-align:center;">{{ chargingTime }}</h4>
      </div>
    </div>
    <div class="form-floating my-4">
      <button type="button" class="btn btn-primary btn-success btn-lg" style="display: block;width: 100%;"
      v-on:click="sendCarRemainingDistance">
        Optimize
        </button>
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
      chargingTime: "",
      sliderContained: false,
      batteryStatus: 10,
      minSlider: 0,
      maxSlider: 100,
      selectedStation: null,
      selectedCar: null,
      chargingStations: null,
      cars: null,
    };
  },
  mounted() {
    this.cars = this.getCars();
    this.selectedCar = this.cars[0];
    this.chargingStations = this.getStations();
    this.selectedStation = this.chargingStations[0];
  },
  methods: {
    sendCarRemainingDistance: function(){
      var remaningDistance = this.selectedCar.range * (this.batteryStatus/100);
      this.$root.$emit("send_carRemainingDistance", remaningDistance, this.selectedCar.range);
    },
    getCars: function () {
      return [
        { name: "Tesla Model S", battery: 100, range: 637},
        { name: "Tesla Model Y", battery: 75, range: 507},
        { name: "Renault Zoe", battery: 52, range: 395},
        { name: "Peugeot e-208", battery: 50, range: 275 },
        { name: "Mercedes Concept EQA", battery: 60, range: 400 },
      ];
    },
    getStations: function () {
      return [
        { name: "Level 1", minPower: 1, maxPower: 3 },
        { name: "Level 2", minPower: 3, maxPower: 20 },
        { name: "Level 3 (DCFC)", minPower: 20, maxPower: 50 },
      ];
    },
    customLabelCar: function (car) {
      return car["name"] + ", battery capacity: " + car["battery"] + " kWh";
    },
    customLabelStation: function (station) {
      return (
        station["name"] +
        ", power: " +
        station["minPower"] +
        "-" +
        station["maxPower"] +
        " kW"
      );
    },
    calcChargingTime: function(){
      var capacity = this.selectedCar.battery;
      var minPower = this.selectedStation.minPower;
      var maxPower = this.selectedStation.maxPower;
      var capacityToFill = capacity - capacity * (this.batteryStatus/100);
      var minTime = capacityToFill/maxPower;
      var maxTime = capacityToFill/minPower;
      this.chargingTime= minTime.toFixed(1) + " - " + maxTime.toFixed(1) + " hours";
    }
  },
};
</script>

<style scoped>
#electric {
  z-index: 10000;
  vertical-align: middle;
}
.slider {
  z-index: 1;
}
.vue-slider-rail {
  z-index: 1;
}
.vue-slider-dot {
  z-index: 1;
}
.vue-slider-dot-hover {
  z-index: 1;
}
.vue-slider-dot-handle {
  z-index: 1;
}
</style>
