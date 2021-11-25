<template>
  <div id="app">
    <Donut :radius="70" :value="value" :maxValue="maxValue"> 25 M </Donut>
  </div>
</template>

<script>
import Donut from './components/Donut';

export default {
  name: 'App',
  data: () => ({
    value: 2.25e6,
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
