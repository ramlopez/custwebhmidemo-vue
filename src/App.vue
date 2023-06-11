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

      // PLC Authentication data. Login hardcoded to default simulation values
      username: "admin",
      password: "plcnext",
      authToken: "",
      accessToken: "",

      // Variable group ID: we can register a list of variables to read easily
      varGroupId: "",
      // Array to hold all of the data we should render on the read table
      readVariables: [
        { varName: "Var 1", dataType: "BOOL", value: true },
        { varName: "Var 2", dataType: "INT", value: 8646 },
        { varName: "Var 3", dataType: "REAL", value: 214.12 },
      ]
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
        // Also register variable group
        .then((rsp) => { this.regGroup() })
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
        // Take group id value and store it in component data
        .then((data) => {
          this.varGroupId = data.id;
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
          this.readVariables = data.variables;
          return this.data;
        })
        // Catch errors, log to console
        .catch((err) => {
          this.accessToken = "";
          console.error("Error reading variable group");
          console.error(err);
        })
    },

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
      <p>Access token:
        <pre>{{ accessToken }}</pre>
      </p>
      <p>Variable group ID:
        <pre>{{ varGroupId }}</pre>
      </p>
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
        <ReadTableRow v-for="v in readVariables" v-bind:var-name="v.path" v-bind:data-type="v.dataType"
          v-bind:raw-value="v.value" />
      </tbody>
    </table>
    <button v-on:click="readGroup()">Update PLC Variables</button>
  </div>
</template>
