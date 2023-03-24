<template>
  <div>
    <div class="container mt-1" >
          <TransformationRow @undo-transformation="undoTransformation"  @apply-transformation="applyTransformation" v-for="(m, index) in meta" v-bind:key="index" :field="m.field" :type="m.type" ></TransformationRow>
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
      },
      applyTransformation(transformation, field, n_characters){
        this.$emit("apply-transformation", transformation, field, n_characters)
      },
      undoTransformation(transformation, field, n_characters){
        this.$emit("undo-transformation", transformation, field, n_characters)
      }
    },
  props:{
      fields: Array,
      displayed: Array
  },
  data(){
    return{
      types: [],
      // Contains informations about the name of the field and the inferred type
      meta: []
    }
  },
  created(){
    // Infer types
    for (const idx in this.fields) {
      let current = {}
      current.type = this.inferType(this.displayed["0"][this.fields[idx]])
      current.field = this.fields[idx]
      this.meta.push(current)
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
