<template>
  <div id="app">
    <Donut
      :radius="70"
      :values="values"
      :maxValue="maxValue"
      isHalf
      showValues
    />
  </div>
</template>

<script>
import Donut from './components/Donut';

export default {
  name: 'App',
  data: () => ({
    values: [0.5e6, 1e6, 0.25e6, 0.5e6, 1e5],
    maxValue: 25e5,
  }),
  components: {
    Donut,
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
  height: 90vh;
}
</style>
