<template>
  <div id="app">
    <router-view
      :isAuthenticated="isAuthenticated"
      :loading="loading"
      :error="error"
      :user="user"
      :cryptrClient="cryptrClient"
      :firstIdp="firstIdp"
      :idpIds="idpIds"
    />
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
      idpIds: [],
      firstIdp: null,
    };
  },
  async created() {
    this.firstIdp = firstIdp;
    this.idpIds = idpIds;
    this.cryptrClient = await CryptrSpa.createClient(config);
    window.addEventListener(
      CryptrSpa.events.REFRESH_INVALID_GRANT,
      (RigError) => {
        console.error(RigError);
        this.cryptrClient.logOut();
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
