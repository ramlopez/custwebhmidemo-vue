<!-- App is our root component of the entire Vue app -->

<!-- Javascript Vue code for this component -->
<script>

// Import Vue components from other files
import ReadTableRow from "./components/ReadTableRow.vue";
import BoolCheckbox from "./components/BoolCheckbox.vue";

// Most of the script code in a Vue component goes in export default
export default {

  // List of imported components
  components: {
    ReadTableRow, BoolCheckbox
  },

  // Data contains reactive variables for this component
  data() {
    return {

      // PLC Authentication data. Login hardcoded to default simulation values
      username: "admin",
      password: "plcnext",
      authToken: "",
      accessToken: "",

      // Variable group ID: we can register a list of variables to read easily
      varGroupId: "",
      // Array to hold path and PLC datatype of registered variables
      groupVariablesRegistration: [
        { path: "Placeholder 1", type: "BOOL" },
        { path: "Placeholder 2", type: "INT" },
        { path: "Placeholder 3", type: "REAL" },
      ],
      // Array to hold the actual read values from the group variables
      groupVariablesValues: [{ value: true }, { value: 546 }, { value: 87.15 }],

      // Ints
      iInt1: 0, iInt2: 0, iInt3: 0, iInt4: 0,
    }
  },

  // Computed properties, derived values based on data that update automatically
  // only when necessary. Useful for transforming raw data
  computed: {

    // Combine groupVariablesRegistration and groupVariablesValues into one
    // array with their data together
    groupVariables() {
      // map() is like a forEach loop, but it calls a function on every element
      return this.groupVariablesRegistration.map((reg, index) => {
        return {
          path: reg.path,
          type: reg.type,
          value: this.groupVariablesValues[index].value,
        }
      })
    }
  },

  // Methods are functions for this component
  methods: {

    // Step 1 in Authentication: Request auth token. A token we use in step 2
    // when making a login attempt
    async reqAuthToken() {
      // Perform POST request to REST API with fetch (async promise)
      return fetch("/_pxc_api/v1.8/auth/auth-token", {
        method: "POST",
        body: JSON.stringify({ "scope": "variables" })
      })
        // Turn response body to JSON (also an async promise)
        .then((rsp) => { return rsp.json() })
        // Take auth token value and store it in component data
        .then((data) => {
          this.authToken = data.code;
          return this.data;
        })
        // Catch errors, log to console
        .catch((err) => {
          this.authToken = "";
          console.error("Error requesting auth token");
          console.error(err);
        })
    },

    // Step 2 in Authentication: Attempt login, request access token. If
    // successful we get an access token we'll need to send with requests that
    // require auth (read/write variables)
    async reqAccessToken() {
      // Perform POST request to REST API with fetch (async promise)
      return fetch("/_pxc_api/v1.8/auth/access-token", {
        method: "POST",
        body: JSON.stringify({
          code: this.authToken,
          grant_type: "authorization_code",
          username: this.username,
          password: this.password
        })
      })
        // Turn response body to JSON (also an async promise)
        .then((rsp) => { return rsp.json() })
        // Take access token value and store it in component data
        .then((data) => {
          this.accessToken = data.access_token;
          return this.data;
        })
        // Catch errors, log to console
        .catch((err) => {
          this.accessToken = "";
          console.error("Error requesting access token");
          console.error(err);
        })
    },

    // Perform both authentication steps in order
    async authenticate() {
      return this.reqAuthToken()
        .then((rsp) => { return this.reqAccessToken() })
        // Also register variable group and read it
        .then((rsp) => { return this.regGroup() })
        .then((rsp) => { return this.readGroup() })
    },

    // Register variable group (a list of variables we register to read easily
    // several times by just using the group ID)
    async regGroup() {
      // Perform POST request to REST API with fetch (async promise)
      return fetch("/_pxc_api/v1.8/groups", {
        method: "POST",
        // We provide access token in header because it requires auth
        headers: { "Authorization": "Bearer " + this.accessToken },
        body: JSON.stringify({
          // We specify the list of variables for the group. We can read any
          // variables that are HMI accessible
          pathPrefix: "Arp.Plc.Eclr/",
          paths: [
            "xBool1",
            "xBool2",
            "xBool3",
            "xBool4",
            "iInt1",
            "iInt2",
            "iInt3",
            "iInt4",
            "rReal1",
            "rReal2",
            "rReal3",
            "rReal4",
          ]
        })
      })
        // Turn response body to JSON (also an async promise)
        .then((rsp) => { return rsp.json() })
        // Take group id and path/datatypes and store those in component data
        .then((data) => {
          this.varGroupId = data.id;
          this.groupVariablesRegistration = data.variables;
          return this.data;
        })
        // Catch errors, log to console
        .catch((err) => {
          this.accessToken = "";
          console.error("Error requesting access token");
          console.error(err);
        })
    },

    // Read registered variables group
    async readGroup() {
      // Perform GET request to REST API with fetch (async promise)
      return fetch("/_pxc_api/v1.8/groups/" + this.varGroupId, {
        method: "GET",
        // We provide access token in header because it requires auth
        headers: { "Authorization": "Bearer " + this.accessToken },
        // Using a group saves us having to list all of them again in request
      })
        // Turn response body to JSON (also an async promise)
        .then((rsp) => { return rsp.json() })
        // Take read variable data and store in component
        .then((data) => {
          this.groupVariablesValues = data.variables;
          return this.data;
        })
        // Catch errors, log to console
        .catch((err) => {
          this.accessToken = "";
          console.error("Error reading variable group");
          console.error(err);
        })
    },

    // Handle boolean checkbox change, write variable to PLC API
    async handleBoolCheckboxChange(eventPayload) {
      // Perform PUT request to REST API with fetch (async promise)
      return fetch("/_pxc_api/v1.8/variables", {
        method: "PUT",
        // We provide access token in header because it requires auth
        headers: { "Authorization": "Bearer " + this.accessToken },
        body: JSON.stringify({
          pathPrefix: "Arp.Plc.Eclr/",
          variables: [
            {
              path: eventPayload.variablePath,
              value: eventPayload.value,
              valueType: "Constant",
            }
          ]
        })
      })
        // Catch errors, log to console
        .catch((err) => {
          console.error("Error writing variable: " + eventPayload.variablePath);
          console.error(err);
        })
    },

    // Write int values from number fields
    async writeInts() {
      // Perform PUT request to REST API with fetch (async promise)
      return fetch("/_pxc_api/v1.8/variables", {
        method: "PUT",
        // We provide access token in header because it requires auth
        headers: { "Authorization": "Bearer " + this.accessToken },
        body: JSON.stringify({
          pathPrefix: "Arp.Plc.Eclr/",
          variables: [
            {
              path: "iInt1",
              value: this.iInt1,
              valueType: "Constant",
            },
            {
              path: "iInt2",
              value: this.iInt2,
              valueType: "Constant",
            },
            {
              path: "iInt3",
              value: this.iInt3,
              valueType: "Constant",
            },
            {
              path: "iInt4",
              value: this.iInt4,
              valueType: "Constant",
            },
          ]
        })
      })
        // Catch errors, log to console
        .catch((err) => {
          console.error("Error writing int variables");
          console.error(err);
        })
    }

  }

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
    <h2>Login</h2>
    <p>We need to log in before we can read or write PLC variables</p>
    <div>
      <!-- v-model makes a two-way binding between these input elements and
        component variables. It combines v-bind + v-on essentially -->
      <label for="username">Username:</label>
      <input v-model="username" type="text" name="username" id="username">
      <label for="password">Password:</label>
      <input v-model="password" type="password" name="password" id="password">
      <button v-on:click="authenticate()">Log in</button>
      <p>Access token: <span class="monospace">{{ accessToken }}</span></p>
      <p>Variable group ID: <span class="monospace">{{ varGroupId }}</span></p>
    </div>
  </div>

  <div>
    <h2>Read PLC Variables</h2>
    <p>
      Read only table showing variables read from the PLC. For this we read
      using a variable group that we register once when we log in, so we don't
      have to re-specify the variable paths every single time we read.
    </p>
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
        <ReadTableRow v-for="v in groupVariables" v-bind:var-name="v.path" v-bind:data-type="v.type"
          v-bind:raw-value="v.value" />
      </tbody>
    </table>
    <button v-on:click="readGroup()">Update PLC Variables</button>
  </div>

  <div>
    <h2>Write PLC values</h2>
    <p>Some example components that write a value to PLC variables</p>

    <div>
      <h3>Checkboxes for booleans</h3>
      <BoolCheckbox v-bind:label="'Bool 1'" v-bind:variable-path="'xBool1'" v-on:chk-change="handleBoolCheckboxChange" />
      <BoolCheckbox v-bind:label="'Bool 2'" v-bind:variable-path="'xBool2'" v-on:chk-change="handleBoolCheckboxChange" />
      <BoolCheckbox v-bind:label="'Bool 3'" v-bind:variable-path="'xBool3'" v-on:chk-change="handleBoolCheckboxChange" />
      <BoolCheckbox v-bind:label="'Bool 4'" v-bind:variable-path="'xBool4'" v-on:chk-change="handleBoolCheckboxChange" />
    </div>

    <div>
      <h3>Normal number fields for ints</h3>
      <div>
        <label for="intInp1">Int 1: </label>
        <input v-model="iInt1" type="number" name="intInp1" id="inptInp1">
      </div>
      <div>
        <label for="intInp2">Int 2: </label>
        <input v-model="iInt2" type="number" name="intInp2" id="inptInp2">
      </div>
      <div>
        <label for="intInp3">Int 3: </label>
        <input v-model="iInt3" type="number" name="intInp3" id="inptInp3">
      </div>
      <div>
        <label for="intInp4">Int 4: </label>
        <input v-model="iInt4" type="number" name="intInp4" id="inptInp4">
      </div>
      <button v-on:click="writeInts">Write Int values</button>
    </div>

  </div>
</template>

<!-- CSS styles that apply globally -->
<style>
/* Reset all styles to box-sizing: border-box */
html {
  box-sizing: border-box;
}

*,
*::before,
*::after {
  box-sizing: inherit;
}

/* Fonts */
html {
  font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
  color: #303030;
}

/* App width and center */
#app {
  max-width: 960px;
  margin-left: auto;
  margin-right: auto;
}
</style>

<!-- Scoped style only applies to this component -->
<style scoped>
/* Table styling */
table {
  border-collapse: collapse;
}

th,
td {
  padding: 4px;
}

th {
  background-color: #f2f2f2;
}

tbody tr:nth-child(even) {
  background-color: #f9f9f9;
}

tbody tr:hover {
  background-color: #e3e3e3;
}

td {
  border: 1px solid #cccccc;
}


/* Monospaced spans */
.monospace {
  font-family: "Courier New", Courier, monospace;
  font-size: 0.9rem;
  background-color: #cccccc;
}
</style>