<template>
  <div class="container my-4">
    <div class="my-4">
      <h5>Relation:</h5>
      <multiselect
        v-model="route"
        :options="routes"
        searchable
        track-by="name"
        :customLabel="customLabelRoute"
        :allow-empty="false"
        placeholder="Select a route"
        @select="routeSelected"
      ></multiselect>
    </div>
    <div>
      <h5>Electric vehicle model:</h5>
      <multiselect
        v-model="selectedCar"
        :options="cars"
        searchable
        track-by="name"
        :customLabel="customLabelCar"
        :allow-empty="false"
        placeholder="Select a car"
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
        :allow-empty="false"
        placeholder="Select charging station type"
      ></multiselect>
    </div>
    <div class="form-floating my-4">
      <h5>Battery fill percentage: {{ batteryStatus }}%</h5>
      <vue-slider
        id="slider"
        v-model="batteryStatus"
        :min="0"
        :max="100"
        :contained="false"
        @change="calcChargingTime"
      >
      </vue-slider>
    </div>
    <div class="grid-container my-4">
      <div class="row">
        <h3 style="vertical-align: middle">Charging time:</h3>
        <h4 class="col" style="text-align: center">{{ chargingTime }}</h4>
      </div>
    </div>
    <div class="form-floating my-4">
      <button
        type="button"
        class="btn btn-primary btn-success btn-lg"
        style="display: block; width: 100%"
        @click="sendCarRemainingDistance"
      >
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
      coffeeIcon: require("@/assets/icons8-coffee-50.png"),
      fuelIcon: require("@/assets/icons8-fuel-64.png"),
      parkingIcon: require("@/assets/icons8-parking-48.png"),
      chargingTime: "",
      batteryStatus: 10,
      selectedStation: null,
      selectedCar: null,
      chargingStations: null,
      cars: null,
      route: null,
      routes: null,
    };
  },
  mounted() {
    this.cars = this.getCars();
    this.selectedCar = this.cars[0];

    this.chargingStations = this.getStations();
    this.selectedStation = this.chargingStations[0];

    this.routes = this.getRoutes();
    this.route = this.routes[0];
  },
  methods: {
    sendCarRemainingDistance() {
      var remainingDistance =
        this.selectedCar.range * (this.batteryStatus / 100);
      this.$root.$emit(
        "send_carRemainingDistance",
        remainingDistance,
        this.selectedCar.range
      );
    },
    routeSelected(selectedRoute) {
      this.$root.$emit("route_selected", selectedRoute);
    },
    getRoutes() {
      return [
        { name: "Zagreb-Split", value: "cite:Zagreb-Split", distance: 410 },
        { name: "Zagreb-Rijeka", value: "cite:Zagreb-Rijeka", distance: 162 },
        { name: "Zagreb-Osijek", value: "cite:Zagreb-Osijek", distance: 284 },
        { name: "Split-Rijeka", value: "cite:Split-Rijeka", distance: 416 },
        { name: "Split-Osijek", value: "cite:Split-Osijek", distance: 690 },
        { name: "Rijeka-Osijek", value: "cite:Rijeka-Osijek", distance: 445 },
      ];
    },
    getCars() {
      return [
        { name: "Tesla Model S", battery: 100, range: 637 },
        { name: "Tesla Model Y", battery: 75, range: 507 },
        { name: "Renault Zoe", battery: 52, range: 395 },
        { name: "Peugeot e-208", battery: 50, range: 275 },
        { name: "Mercedes Concept EQA", battery: 60, range: 400 },
      ];
    },
    getStations() {
      return [
        { name: "Level 1", minPower: 1, maxPower: 3 },
        { name: "Level 2", minPower: 3, maxPower: 20 },
        { name: "Level 3 (DCFC)", minPower: 20, maxPower: 50 },
      ];
    },
    customLabelCar(car) {
      return car
        ? `${car.name}, battery capacity: ${car.battery} kWh, range: ${car.range} km`
        : "";
    },
    customLabelStation(station) {
      if (station == "") return "";
      return (
        station["name"] +
        ", power: " +
        station["minPower"] +
        "-" +
        station["maxPower"] +
        " kW"
      );
    },
    customLabelRoute(selectedRoute) {
      return selectedRoute
        ? `${selectedRoute.name}, distance: ${selectedRoute.distance} km`
        : "";
    },
    calcChargingTime() {
      var capacity = this.selectedCar.battery;
      var minPower = this.selectedStation.minPower;
      var maxPower = this.selectedStation.maxPower;
      var capacityToFill = capacity - capacity * (this.batteryStatus / 100);
      var minTime = capacityToFill / maxPower;
      var maxTime = capacityToFill / minPower;
      this.chargingTime =
        minTime.toFixed(1) + " - " + maxTime.toFixed(1) + " hours";
    },
  },
};
</script>

<style>
.multiselect--active {
  z-index: 10;
}
</style>
