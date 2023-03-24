<template>
  <div>
    <div class="container mt-1" >
          <TransformationRow v-for="(m, index) in meta" v-bind:key="index" :field="m.field" :type="m.type" ></TransformationRow>
    </div>
  </div>
</template>

<script>
import TransformationRow from './TransformationRow.vue'


export default {
  name: 'TransformationsPanel',
  components: {
    TransformationRow
  },
  methods: {
      inferType(sample){
        let parsed_sample = parseFloat(sample)
        let parsed_type
        if (!isNaN(parsed_sample)){
          parsed_type = "float"
        }
        else {
          parsed_type = "string"
        }
        return parsed_type
      }
    },
  props:{
      fields: Array,
      displayed: Array
  },
  data(){
    return{
      displayed: [],
      fiedl: []
    }
  },
  created(){
    // Infer types
    for (const idx in this.fields) {
      let current = {}
      current.type = this.inferType(this.displayed["0"][this.fields[idx]])
      current.field = this.fields[idx]
      console.log(current)
      this.meta.push(current)
    }
    console.log(this.meta)
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
