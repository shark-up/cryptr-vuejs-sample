<template>
  <div id="app">
    <!-- <div id="nav">
      <router-link to="/">Home</router-link> |
      <router-link to="/private">Private</router-link>
    </div> -->
    <div>
      <span>{{ error }}</span>
      <span v-if="!loading">Authenticated: {{ isAuthenticated }}</span>
      <span v-if="loading">Loading ..</span>
      <button
        v-if="isAuthenticated"
        type="button"
        class="btn btn-danger"
        @click="logOut"
      >
        Logout
      </button>
      <div class="pricing-header px-3 py-3 pt-md-5 pb-md-4 mx-auto text-center">
        <h1 class="display-4">Development mode</h1>
        <p class="lead">
          This local static site use our cryptr API on our test environment.
        </p>
        <button
          href="#"
          class="btn btn-secondary"
          v-if="!loading"
          @click="signinWithSSO"
        >
          Signin with SSO
        </button>
      </div>
      <div class="card mb-4 box-shadow">
        <div class="card-header">
          <h4 class="my-0 font-weight-normal">Gateway</h4>
          <p>You don't known what's user's IDP? use our gateway</p>
        </div>
        <div class="card-body" v-if="!loading">
          <button
            type="button"
            @click="globalGateway"
            class="btn btn-info mr-3 btn-sm"
            href="#"
          >
            Global Gateway
          </button>
          <button
            type="button"
            @click="gatewayIdp"
            class="btn btn-info mr-3 btn-sm"
            href="#"
          >
            Gateway with one IDP
          </button>
          <button
            type="button"
            @click="gatewayIdps"
            class="btn btn-info mr-3 btn-sm"
            href="#"
          >
            Gateway with multiple idps
          </button>
        </div>
      </div>
      <div v-if="user.email" class="card box-shadow">
        <div class="card-header">
          <h4>User</h4>
        </div>
        <div class="card-body" v-if="!loading">
          <p v-if="user.ips">
            You connected through {{ user.ips }} provider (using '{{ user.sci }}'
            idp)
          </p>
          <ul>
            <li>Email: {{ user.email }}</li>
            <li>User ID: {{ user.sub }}</li>
            <li>Organization Domain: {{ user.tnt }}</li>
          </ul>
        </div>
      </div>
    </div>
    <router-view :isAuthenticated="isAuthenticated" />
  </div>
</template>

<style>
@import "./assets/css/argon-dashboard.css";
@import "./assets/css/nucleo-icons.css";
@import "./assets/css/nucleo-svg.css";
@import "https://kit.fontawesome.com/42d5adcbca.js";
</style>

<script>
import CryptrSpa from "@cryptr/cryptr-spa-js";
var firstIdp = null;
var idpIds = null;
if (process.env.VUE_APP_CRYPTR_IDP_IDS) {
  idpIds = process.env.VUE_APP_CRYPTR_IDP_IDS.split(",");
  firstIdp = idpIds[0];
} else {
  console.warn("no VUE_APP_CRYPTR_IDP_IDS");
}
console.log(firstIdp);
console.log(idpIds);
const config = {
  tenant_domain: process.env.VUE_APP_CRYPTR_TENANT_DOMAIN,
  client_id: process.env.VUE_APP_CRYPTR_CLIENT_ID,
  audience: process.env.VUE_APP_CRYPTR_AUDIENCE,
  default_redirect_uri: process.env.VUE_APP_CRYPTR_REDIRECT_URI,
  cryptr_base_url: process.env.VUE_APP_CRYPTR_BASE_URL,
  default_locale: process.env.VUE_APP_CRYPTR_DEFAULT_LOCALE,
  telemetry: process.env.VUE_APP_CRYPTR_TELEMETRY === "true",
  dedicated_server: process.env.VUE_APP_CRYPTR_DEDICATED_SERVER === "true",
};

export default {
  data() {
    return {
      loading: false,
      isAuthenticated: false,
      user: {},
      cryptrClient: null,
      error: null,
    };
  },
  methods: {
    globalGateway() {
      this.cryptrClient.signInWithSSOGateway();
    },
    gatewayIdp() {
      this.cryptrClient.signInWithSSOGateway(firstIdp);
    },
    gatewayIdps() {
      this.cryptrClient.signInWithSSOGateway(idpIds);
    },
    signinWithSSO() {
      return this.cryptrClient.signInWithSSO(firstIdp);
    },
    logOut() {
      return this.cryptrClient.logOut(() => {
        alert("You are now logged out ! Bye :(");
        window.location.reload();
        window.location.replace(location.href.split("?")[0]);
      });
    },
  },
  async created() {
    this.cryptrClient = await CryptrSpa.createClient(config);
    window.addEventListener(
      CryptrSpa.events.REFRESH_INVALID_GRANT,
      (RigError) => {
        console.error(RigError);
        this.logOut();
      }
    );

    try {
      this.loading = true;
      const canAuthenticate = await this.cryptrClient.canHandleAuthentication();
      if (canAuthenticate) {
        const process = await this.cryptrClient.handleRedirectCallback();
        console.debug("process redirect callback");
        console.debug(process);
        let params = new URL(window.location).searchParams;
        let authParamsKeys = [
          "authorization_code",
          "authorization_id",
          "state",
          "code",
          "organization_domain",
        ];
        for (const key of authParamsKeys) {
          params.delete(key);
        }
        const endUrl = params.toString() == "" ? "" : "?" + params.toString();

        window.history.replaceState(
          {},
          document.title,
          window.location.pathname + endUrl
        );
      } else {
        await this.cryptrClient.handleRefreshTokens();
      }
    } catch (e) {
      this.error = e;
    } finally {
      this.isAuthenticated =
        await this.cryptrClient.currentAccessTokenPresent();
      if (this.cryptrClient.getUser()) {
        this.user = this.cryptrClient.getUser();
      } else {
        this.user = {};
      }
      this.loading = false;
    }
  },
};
</script>
