<template>
  <div class="container mt-3">
    <div class="row dropdown">
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
    <div class="row dropdown">
       <multiselect
        v-model="selectedStation"
        :options="chargingStations"
        searchable
        track-by="name"
        label="name"
        allow-empty
        placeholder="Select a charging station"
      ></multiselect>
      
    </div>
  </div>
</template>

<script>

export default {
  name: "ConfigurationForm",
  data() {
    return {
      selectedCar: null,
      selectedStation: null,
      cars: null,
      chargingStations: null,
    };
  },
  mounted() {
    this.cars = this.getCars();
    this.chargingStations = this.getChargingStations();
    this.selectedCar = this.cars[0];
    this.selectedStation = this.chargingStations[0];
  },
  methods: {
    getCars: function () {
      return [{ name: "fiesta", batery: 100 }, {name: "punto", batery: 0}, {name: "twingo", batery: 10000}];
    },
    getChargingStations: function () {
      return [{ name: "HEP", power: 100 }, {name: "DINAMO", power: 10000}, {name: "BUREK", power: 100000}];
    },
  },
};
</script>

<style scoped></style>
