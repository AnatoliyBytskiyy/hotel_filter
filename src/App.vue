<template>
  <div id="app">
    <HotelView msg="Welcome to Your Vue.js App"/>
  </div>
</template>

<script>
import HotelView from './components/HotelView.vue'

export default {
  name: 'App',
  components: {
    HotelView
  }
}
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: left;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
