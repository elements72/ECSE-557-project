<template>
  <div class="">
        <form class="form-inline row">
        <div class="form-group mb-2 col-5">
            <label class="sr-only">{{ field }}</label>
        </div>
        <div class="form-group col-4">
            <select class="form-select" >
            <option>Drop</option>
            <option v-for="(trans, index) in chooseTransormation()" v-bind:key="index">{{ trans }}</option>
            </select>
        </div>
        <div class="col-3">
            <button type="button" class="btn btn-primary mb-2 col">Apply</button>
            <button type="button" class="btn btn-danger mb-2 col ml-2">Undo</button>
        </div>
        </form>        
  </div>
</template>

<script>

export default {
  name: 'TransformationRow',
  components: {
  },
  methods: {
    chooseTransormation(){
        if(this.current_type == "string"){
            return this.str_transformations
        }
        else if (this.current_type == "int"){
            return this.int_transformations
        }
        else {
            return this.float_transformations
        }
    }
    },
  props:{
      field: String,
      type: String
  },
  data() {
    return {
        str_transformations: ["End cropping", "Start cropping"],
        int_transformations: ["Rounding", "Grouping"],
        float_transformations: ["Rounding"],
        current_type: String,
    }
  },
  created(){
    this.current_type = this.type
    console.log("Field in row " + this.field)
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
