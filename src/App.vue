<template>
  <div id="app">
    <div class="container">
      <world-time />
    </div>
  </div>
</template>

<script>
import WorldTime from './components/WorldTime.vue'

export default {
  name: 'App',
  components: {
    WorldTime
  }
}
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.container {
  max-width: 400px;
  margin: auto;
  padding: 0 10px;
}
</style>
