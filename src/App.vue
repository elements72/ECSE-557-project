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

      insertAt(array, index, ...elementsArray) {
        array.splice(index, 0, ...elementsArray);
      },

      dropUndo(field){
        let idx_feature
        for (const idx in this.fields_removed) {
          if (this.fields_removed[idx]["field"] == field){
            idx_feature = this.fields_removed[idx]["idx"]
            this.fields_removed.splice(idx, 1)
            break;
          }
        }
        // Remove the field from the removed ones
        const index = this.fields_removed.indexOf(field);
        this.fields_removed.splice(index, 1);
        console.log("Inserting at index ", idx_feature  )
        this.insertAt(this.fields_transformed, idx_feature, field);
      },

      cropping(data, field, n_characters, start){
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

      findMinMax(data, field) {
        let min = Infinity
        let max = - Infinity
        for (const element of data){
          let value = parseFloat(element[field]) 
          if(value < min){
            min = value
          }
          else if (value > max){
            max = value
          }
        }
        return {"min": min, "max": max}
      },
      // Create the bins for grouping transformation
      createBins(data, field, interval) {
        let bins = [];
        let binCount = 0;
        let numOfBuckets;
        interval = parseInt(interval)

        let minMax = this.findMinMax(data, field)
        let min = minMax.min
        let max = minMax.max

        min = parseInt(this.zeroReplace(min.toString()))

        numOfBuckets = Math.ceil((max - min) / interval)

        // Set up the first bin
        bins.push({
            binNum: binCount,
            minNum: min,
            maxNum: min + interval,
            count: 0
        })
        binCount++
        //Setup Bins
        for(let i = 1; i < numOfBuckets; i += 1){
          let minNum = bins[i-1].maxNum
          bins.push({
            binNum: binCount,
            minNum: minNum,
            maxNum: minNum + interval,
            count: 0
          })
          binCount++;
        }
        return bins
      },

      grouping(data, field, bins){
        //Loop through data and add to bin's count
        for (var i = 0; i < data.length; i++){
          var item = data[i][field];
          for (var j = 0; j < bins.length; j++){
            var bin = bins[j];
            if(item >= bin.minNum && item <= bin.maxNum){
              bin.count++;
              // TODO - do not display the real number - just for testing purposes
              //data[i][field] = `${bin.minNum}<${item}<=${bin.maxNum}`
              data[i][field] = `${bin.minNum}-${bin.maxNum}`
              break;  // An item can only be in one bin.
            }
          }  
        }
      },
      undoGrouping(field){
        for (const idx in this.grouping_applied) {
          if (this.grouping_applied[idx]["field"] == field){
            this.grouping_applied.splice(idx, 1)
          }
        }
        for (const idx in this.displayed_transformed) {
          this.displayed_transformed[idx][field] = this.displayed[idx][field]
        }
      },

      rounding(data, field, precision){
        for (let element of data) {
          element[field] = Math.round(parseFloat(element[field]), precision)
        }
      },

      undoRounding(field){
        for (const idx in this.rounding_applied) {
          if (this.rounding_applied[idx]["field"] == field){
            this.rounding_applied.splice(idx, 1)
          }
        }
        for (const idx in this.displayed_transformed) {
          this.displayed_transformed[idx][field] = this.displayed[idx][field]
        }
      },

      zeroReplace(value){
        let counter = 0
          for (let i = 1; i < value.length; i++) {
            if(value[i] == ',')
              break
            else
              counter++
          }
           return value[0] + "0".repeat(counter)
        },

      powerRounding(data, field){
        for (let idx in data) {
          data[idx][field] = this.zeroReplace(data[idx][field])
        }
      },

      undoPowerRounding(field){
        for (const idx in this.power_rounding_applied) {
          if (this.power_rounding_applied[idx]["field"] == field){
            this.power_rounding_applied.splice(idx, 1)
          }
        }
        for (const idx in this.displayed_transformed) {
          this.displayed_transformed[idx][field] = this.displayed[idx][field]
        }
      },


      applyTransformation(transformation, field, size){
        switch (transformation) {
          case "Drop":
            console.log(this.fields.indexOf(field))
            this.fields_removed.push({"field": field, "idx": this.fields.indexOf(field)})
            this.drop(field)
            break;
          case "Start cropping":
            this.cropping_applied.push({"field": field, "n_characters": size, "start": true})
            this.cropping(this.displayed_transformed, field, size, true)
            break;  
          case "End cropping":
            this.cropping_applied.push({"field": field, "n_characters": size, "start": false})
            this.cropping(this.displayed_transformed, field, size, false)
            break
          case "Grouping": {
            // Create the bins
            let bins = this.createBins(this.content.data, field, size)
            this.grouping_applied.push({"field": field, "bins": bins})
            this.grouping(this.displayed_transformed, field, bins)
            break;
          }
          case "Rounding":
            this.rounding_applied.push({"field": field, "precision": size})
            this.rounding(this.displayed_transformed, field, size)
            break;
          case "Power rounding":
            this.power_rounding_applied.push({"field": field})
            this.powerRounding(this.displayed_transformed, field, size)
            break;
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
          case "Grouping":
            this.undoGrouping(field)
            break;
          case "Rounding":
            this.undoRounding(field)
            break;
          case "Power rounding":
            this.undoPowerRounding(field)
            break;
          default:
            break;
        }
      },
      exportCsv(){
        let exportContent = structuredClone(this.content)
        // Apply droppping
        for (const key in this.fields_removed) {
          let index = exportContent.meta.fields.indexOf(key)
          exportContent.meta.fields.splice(index)
        }
        // Apply cropping
        console.log(this.cropping_applied)
        for (const cropping of this.cropping_applied) {
          this.cropping(exportContent.data, cropping.field, cropping.n_characters, cropping.start)
        }

        // Apply grouping
        for (const grouping of this.grouping_applied) {
          this.grouping(exportContent.data, grouping.field, grouping.bins)
        }

        // Apply rounding
        for (const round of this.rounding_applied) {
          this.rounding(exportContent.data, round.field, round.precision)
        }

        // Apply power rounding
        for (const round of this.power_rounding_applied) {
          this.powerRounding(exportContent.data, round.field)
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
          // Cropping applied
          cropping_applied: [],
          // Groupig applied
          grouping_applied: [],
          // Rounding applied
          rounding_applied: [],
          // Rounding applied
          power_rounding_applied: []
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
