<template>
  <div>
    <div class="overflow-auto">
      <table style="width: 100%;">
        <thead>
            <tr>
                <th v-for="(header, key) in fields"
                    v-bind:key="'header-'+key">
                    {{ header }}
                </th>
            </tr>
        </thead>
        <tbody>
            <tr v-for="(row, rowKey) in displayed"
                v-bind:key="'row-'+rowKey">
                    <td v-for="(column, columnKey) in fields"
                        v-bind:key="'row-'+rowKey+'-column-'+columnKey">
                            <input class="" :value="displayed[rowKey][column]" disabled/>
                    </td>
            </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>

export default {
  name: 'CsvViewer',
  components: {
  },
  methods: {
    },
  props:{
      fields: Array,
      displayed: Array
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
