<template>
  <div class="container">
    <div v-if="$store.state.auth">
      <h1>I'm in</h1>
    </div>
    <p v-else>
      <h1>Sign in to access the secret page</h1>
    <div>
      <label for="email">
        <input id="email" type="email" value="test">
      </label>
      <label for="password">
        <input id="password" type="password" value="test">
      </label>
      <button @click="postLogin">login</button>
      <p>The credentials are not verified for the example purpose.</p>
    </div>
    </p>
  </div>
</template>

<script>
const Cookie = process.client ? require("js-cookie") : undefined;
const Keycloak = require("keycloak-js");
export default {
  middleware: "notAuthenticated",
  methods: {
    postLogin() {
      var config = {
        realm: "bimap",
        url: "https://sso.tokensandbox.io/auth",
        "ssl-required": "external",
        clientId: "ipoc2.0",
        credentials: "75766c85-58e1-4c4f-86a5-67f09c7b23ad",
        "public-client": true
      };
      var keycloak = Keycloak(config);
      console.log(keycloak);

      keycloak.onAuthError = function(errorData) {
        console.log("Auth Error: " + JSON.stringify(errorData));
      };

      keycloak.onAuthRefreshSuccess = function() {
        console.log("Auth Refresh Success");
      };

      keycloak.onAuthRefreshError = function() {
        console.log("Auth Refresh Error");
      };

      keycloak.onAuthLogout = function() {
        console.log("Auth Logout");
      };

      keycloak.onTokenExpired = function() {
        console.log("Access token expired.");
      };

      keycloak.onAuthSuccess = function() {
        console.log("Auth Success");
        setTimeout(() => {
          // we simulate the async request with timeout.
          const auth = {
            accessToken: "someStringGotFromApiServiceWithAjax"
          };
          this.$store.commit("setAuth", auth); // mutating to store for client rendering
          Cookie.set("auth", auth); // saving token in cookie for server rendering
          this.$router.push("/");
        }, 1000);
      };
      // Flow can be changed to 'implicit' or 'hybrid', but then client must enable implicit flow in admin console too
      var initOptions = {
        responseMode: "fragment",
        flow: "implicit"
      };
      keycloak
        .init(initOptions)
        .success(function(authenticated) {
          console.log(
            "Init Success (" +
              (authenticated ? "Authenticated" : "Not Authenticated") +
              ")"
          );
        if (authenticated) {
          console.log(authenticated)
          const auth = {
            accessToken: "someStringGotFromApiServiceWithAjax"
          };
          this.$store.commit("setAuth", auth); // mutating to store for client rendering
          Cookie.set("auth", auth); // saving token in cookie for server rendering
          this.$router.push("/secret");
        }}
        )
        .error(function() {
          console.log("Init Error");
        });

      keycloak.login();
    }
  }
};
</script>
