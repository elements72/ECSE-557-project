<template>
  <div class="">
        <form class="form-inline row">
        <div class="form-group mb-2 col-5">
            <label class="sr-only">{{ field }}</label>
        </div>
        <div class="form-group col-4">
            <select class="form-select" v-model="selected" :disabled="applied">
            <option value="Drop" selected>Drop</option>
            <option :value=trans v-for="(trans, index) in chooseTransormation()" v-bind:key="index">{{ trans }}</option>
            </select>
            <div v-if="selected === 'End cropping' || selected === 'Start cropping'">
              <label for="n_characters">Number of characters</label>
              <input type="number" id="n_characters" min="1" v-model="size" :disabled="applied">
            </div>
            <div v-if="selected === 'Grouping'">
              <label for="bin_size">Bin size</label>
              <input type="number" id="bin_size" min="1" v-model="size" :disabled="applied">
            </div>
            <div v-if="selected === 'Rounding'">
              <label for="precision">Precision</label>
              <input type="number" id="precision" min="1" v-model="size" :disabled="applied">
            </div>
        </div>
        <div class="col-3">
            <button type="button" :disabled="applied" @click="applyTransformation()" class="btn btn-primary mb-2 col">Apply</button>
            <button type="button" :disabled="!applied" @click="undoTransformation()" class="btn btn-danger mb-2 col ml-2">Undo</button>
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
      },
      applyTransformation(){
          this.applied = true
          this.$emit("apply-transformation", this.selected, this.field, this.size);
      },
      undoTransformation(){
          this.applied = false
          this.$emit("undo-transformation", this.selected, this.field, this.size);
      }

    },
  props:{
      field: String,
      type: String
  },
  data() {
    return {
        str_transformations: ["End cropping", "Start cropping"],
        int_transformations: ["Start cropping", "End cropping", "Rounding", "Grouping"],
        float_transformations: ["Start cropping", "End cropping", "Rounding", "Grouping", "Power rounding"],
        current_type: String,
        applied: false,
        selected: "Drop",
        size: 1,  
        bin_size: 10,
    }
  },
  created(){
    this.current_type = this.type
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
