<template>
  <div id="app">
    <OptimalRoute />
  </div>
</template>

<script>
import OptimalRoute from "./components/OptimalRoute.vue";

export default {
  name: "App",
  components: {
    OptimalRoute,
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
body {
  overflow-y: clip;
}
</style>
