<template>
  <div>
    <table v-if="parsed" style="width: 100%;">
      <thead>
          <tr>
              <th v-for="(header, key) in content.meta.fields"
                  v-bind:key="'header-'+key">
                  {{ header }}
              </th>
          </tr>
      </thead>
      <tbody>
          <tr v-for="(row, rowKey) in content.data"
              v-bind:key="'row-'+rowKey">
                  <td v-for="(column, columnKey) in content.meta.fields"
                      v-bind:key="'row-'+rowKey+'-column-'+columnKey">
                          <input v-model="content.data[rowKey][column]" disabled/>
                  </td>
          </tr>
      </tbody>
    </table>
    <input type="file" accept=".csv" @change="handleFileUpload( $event )"/>
  </div>
</template>

<script>

import Papa from 'papaparse';

export default {
  name: 'CsvViewer',
  components: {
  },
  methods: {
    parseFile(){
    Papa.parse( this.file, {
        header: true,
        skipEmptyLines: true,
        complete: function( results ){
            this.content = results;
            this.parsed = true;
        }.bind(this)
      } );
    },
    shuffle_select(){

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
          parsed: false
      }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
