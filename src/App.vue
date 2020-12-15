<template>
  <div id="app">
    <webcamApp/>
    
  </div>
</template>

<script>
import webcamApp from "./components/webcam_app.vue"

export default {

name: 'App',
  components: {
    webcamApp
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
