<template>
  <div>
      <CsvViewer> </CsvViewer>
  </div>
</template>

<script>

import CsvViewer from './components/CsvViewer.vue'

export default {
  name: 'App',
  components: {
    CsvViewer
  },
  methods: {
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
