<template>
  <div class="container">
      <h1 class="h1">Anonymizer tool 1.0</h1>
      <CsvViewer class="mt-5" v-if="parsed" :displayed="displayed" :fields="fields"> </CsvViewer>
      <input class="btn" type="file" accept=".csv" @change="handleFileUpload( $event )"/>
      <TransformationsPanel @undo-transformation="undoTransformation" @apply-transformation="applyTransformation" class="" v-if="parsed" :displayed="displayed" :fields="fields"></TransformationsPanel>
      <CsvViewer class="mt-5" v-if="parsed" :displayed="displayed_transformed" :fields="fields_transformed"> </CsvViewer>
      <input class="btn" type="btn" @click="exportCsv( $event )" value="Export"/>
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
              this.fields = this.content.meta.fields
              this.parsed = true
              this.transformed = this.content
              // Make a copy of the original data
              this.displayed_transformed = structuredClone(this.displayed)
              this.fields_transformed = structuredClone(this.fields)
              console.log(this.content)
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

      drop(field){
        const index = this.fields_transformed.indexOf(field);
        this.fields_transformed.splice(index, 1);
        this.fields_removed.push(field)
      },

      dropUndo(field){
        // Remove the field from the removed ones
        const index = this.fields_removed.indexOf(field);
        this.fields_removed.splice(index, 1);

        this.fields_transformed.push(field);
      },

      cropping(data, field, n_characters, start){
        console.log("Cropping ", field)
        if (start){
          for (const element of data) {
            element[field] = element[field].substring(n_characters, element[field].length)
          }
        }
        else {
            for (const element of data) {
              element[field] = element[field].substring(0, element[field].length-n_characters)
          }
        }
      },

      undoCropping(field){
        for (const idx in this.cropping_applied) {
          if (this.cropping_applied[idx]["field"] == field){
            this.cropping_applied.splice(idx, 1)
          }
        }
        for (const idx in this.displayed_transformed){
          this.displayed_transformed[idx][field] = this.displayed[idx][field] 
        }
      },

      applyTransformation(transformation, field, n_characters){
        switch (transformation) {
          case "Drop":
            this.drop(field)
            break;
          case "Start cropping":
            this.cropping_applied.push({"field": field, "n_characters": n_characters, "start": true})
            this.cropping(this.displayed_transformed, field, n_characters, true)
            break;
          case "End cropping":
            this.cropping_applied.push({"field": field, "n_characters": n_characters, "start": false})
            this.cropping(this.displayed_transformed, field, n_characters, false)
            break
          default:
            break;
        }
      },

      undoTransformation(transformation, field){
        switch (transformation) {
          case "Drop":
            this.dropUndo(field)
            break;
          case "Start cropping":
            this.undoCropping(field)
            break;
          case "End cropping":
            this.undoCropping(field)
            break;
          default:
            break;
        }
      },
      exportCsv(){
        let exportContent = structuredClone(this.content)
        for (const key in this.fields_removed) {
          let index = exportContent.meta.fields.indexOf(key)
          exportContent.meta.fields.splice(index)
        }
        console.log(exportContent.data)
        // Apply cropping
        console.log(this.cropping_applied)
        for (const cropping of this.cropping_applied) {
          this.cropping(exportContent.data, cropping.field, cropping.n_characters, cropping.start)
        }
        var csv = Papa.unparse(exportContent);
        var csvData = new Blob([csv], {type: 'text/csv;charset=utf-8;'});
        var csvURL = window.URL.createObjectURL(csvData);
        var testLink = document.createElement('a');
        testLink.href = csvURL;
        testLink.setAttribute('test', 'test.csv');
        testLink.click();
      }
  },
  data() {
      return {
          file: '',
          content: [],
          parsed: false,
          displayed: [],
          fields: [],
          transformed: [],
          fields_transformed: [],
          displayed_transformed: [],
          // Removed fields
          fields_removed: [],
          cropping_applied: []
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
