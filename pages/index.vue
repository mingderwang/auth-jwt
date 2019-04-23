<template>
  <div>
    <div v-if="$store.state.auth">
      <p>
        You are authenticated. You can see the
        <NuxtLink to="/secret">secret page</NuxtLink>!
      </p>
      <button @click="logout">Logout</button>
    </div>
    <p v-else>
      Please
      <NuxtLink to="/login">login</NuxtLink>.
      <button @click="logout">Logout</button>
    </p>
  </div>
</template>

<script>
const Cookie = process.client ? require("js-cookie") : undefined;
const Keycloak = require("keycloak-js");
export default {
  middleware: "notAuthenticated",
  methods: {
    logout() {
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
          if (!authenticated) {
            keycloak.logout();
            Cookie.remove("auth");
            this.$store.commit("setAuth", null);
          }
        })
        .error(function() {
          console.log("Init Error");
        });
    }
  }
};
</script>
