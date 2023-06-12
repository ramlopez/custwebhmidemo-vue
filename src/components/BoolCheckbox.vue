<!-- This component is a single row in the read-only table for PLC variables -->

<!-- Javascript Vue code for this component -->
<script>

// Most of the script code in a Vue component goes in export default
export default {

  // Props are custom attributes for a component. Parent component instantiating
  // this one can pass values for these
  props: {
    label: String,
    variablePath: String,
  },

  // Data contains reactive variables for this component
  data() {
    return {
      checked: false,
    }
  },

  // Methods are functions for this component
  methods: {
    // Called when checkbox changes, emit event to parent component
    handleChkChange() {
      this.$emit("chk-change", {
        variablePath: this.variablePath,
        value: this.checked,
      })
    }
  }

}

</script>

<!-- Template contains the HTML for the component, and also allows additional
Vue functions to make it reactive and interact with variables from script -->
<template>
  <div class="checkbox-container">
    <!-- v-on="$listeners" is a shorthand for the component to just re-emit the
      events from this element. -->
    <input v-model="checked" v-on:change="handleChkChange" type="checkbox" name="chk" id="chk">
    <label for="chk">{{ label }}</label>
  </div>
</template>

<style scoped>
.checkbox-container {
  background-color: #cccccc;
  display: inline-block;
  padding: 4px;
  margin: 4px;
  border: 1px solid #505050
}

input#chk {
  width: 24px;
  height: 24px;
  vertical-align: middle;
}

label {
  vertical-align: middle;
}
</style>