<template>
  <div class="container">
      <h1 class="h1">Anonymizer tool 1.0</h1>
      <CsvViewer class="mt-5" v-if="parsed" :displayed="displayed" :fields="fields"> </CsvViewer>
      <input class="btn" type="file" accept=".csv" @change="handleFileUpload( $event )"/>
      <TransformationsPanel class="" v-if="parsed" :displayed="displayed" :fields="fields"></TransformationsPanel>
  </div>
</template>

<script>

import CsvViewer from './components/CsvViewer.vue'
import TransformationsPanel from './components/TransformationsPanel.vue'

import Papa from 'papaparse';


export default {
  name: 'App',
  components: {
    CsvViewer,
    TransformationsPanel
  },
  methods: {
   parseFile(){
      Papa.parse( this.file, {
          header: true,
          skipEmptyLines: true,
          complete: function( results ){
              this.content = results;
              this.displayed =  this.shuffle_select(5)
              console.log(this.displayed)
              this.fields = this.content.meta.fields
              this.parsed = true;
          }.bind(this)
        } );
      },
      shuffle_select(N){
        const shuffled = [...this.content.data].sort(() => 0.5 - Math.random());
        return shuffled.slice(0, N);
      },
      handleFileUpload( event ){
      this.file = event.target.files[0];
      this.parseFile();
      },
  },
  data() {
      return {
          file: '',
          content: [],
          parsed: false,
          displayed: [],
          fields: []
      }
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
