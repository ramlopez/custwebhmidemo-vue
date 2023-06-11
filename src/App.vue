<!-- App is our root component of the entire Vue app -->

<!-- Javascript Vue code for this component -->
<script>

// Import Vue components from other files
import ReadTableRow from "./components/ReadTableRow.vue";

// Most of the script code in a Vue component goes in export default
export default {

  // List of imported components
  components: {
    ReadTableRow
  },

  // Data contains reactive variables for this component
  data() {
    return {

      // Array to hold all of the data we should render on the read table
      readVariables: [
        { varName: "Var 1", dataType: "BOOL", value: true },
        { varName: "Var 2", dataType: "INT", value: 8646 },
        { varName: "Var 3", dataType: "REAL", value: 214.12 },
      ]
    }
  },

}

</script>

<!-- Template contains the HTML for the component, and also allows additional
Vue functions to make it reactive and interact with variables from script -->
<template>
  <h1>Vue Web App</h1>
  <p>
    This is an example webpage made with Vue showing how it could be used to
    make a web HMI: uploading the page to the PLCNext controller itself,
    using its webserver to serve it to web browsers acting as HMI, using the
    REST API to read/write PLC Variables.
  </p>
  <div>
    <h2>Read PLC Variables</h2>
    <p>Read only table showing variables read from the PLC</p>
    <table>
      <thead>
        <tr>
          <th>PLC Variable</th>
          <th>PLC Data Type</th>
          <th>Value (raw)</th>
        </tr>
      </thead>
      <tbody>
        <!-- Components are instantiated like a custom HTML element -->
        <!-- v-for is like a foreach loop to render several of a component -->
        <!-- v-bind binds an HTML attribute (or component prop) to a variable
          from this one -->
        <ReadTableRow v-for="v in readVariables"
          v-bind:var-name="v.varName"
          v-bind:data-type="v.dataType"
          v-bind:raw-value="v.value"        
        />
      </tbody>
    </table>
  </div>
  <div></div>
</template>
