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
  }

}

</script>

<!-- Template contains the HTML for the component, and also allows additional
Vue functions to make it reactive and interact with variables from script -->
<template>
  <h1>Vue Web App</h1>
  <p>
    This is an example webpage made with Vue showing how it could be used to
    make a modern web HMI: uploading the page to the PLCNext controller itself,
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
        <ReadTableRow/>
      </tbody>
    </table>
  </div>
  <div></div>
</template>
