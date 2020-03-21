<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png">
    <TCS msg="Welcome to TCS bot"/>
  </div>
</template>

<script>
import TCS from './components/tcs.vue'

export default {
  name: 'App',
  components: {
    TCS
  }
}
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
</style>
