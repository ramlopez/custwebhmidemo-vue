<!-- This component is a single row in the read-only table for PLC variables -->

<!-- Javascript Vue code for this component -->
<script>

// Most of the script code in a Vue component goes in export default
export default {

  // Props are custom attributes for a component. Parent component instantiating
  // this one can pass values for these
  props: {
    varName: String,
    dataType: String,
    rawValue: String
  }

}

</script>

<!-- Template contains the HTML for the component, and also allows additional
Vue functions to make it reactive and interact with variables from script -->
<template>
<tr>
  <td>{{ varName }}</td>
  <td>{{ dataType }}</td>
  <td>{{ rawValue }}</td>
</tr>
</template>
