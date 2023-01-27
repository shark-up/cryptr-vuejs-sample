<template>
  <div id="app">
    <router-view
      :isAuthenticated="isAuthenticated"
      :loading="loading"
      :error="error"
      :user="user"
      :cryptrClient="cryptrClient"
      :adfsIdp="adfsIdp"
      :firstIdp="firstIdp"
      :idpIds="idpIds"
      :firstDomain="firstDomain"
      :secondDomain="secondDomain"
      :idpByProvider="idpByProvider"
      :magicLinkActivated="magicLinkActivated"
    />
  </div>
</template>

<style>
/* @import "./assets/css/argon-dashboard.css"; */
@import "./assets/css/nucleo-icons.css";
@import "./assets/css/nucleo-svg.css";
@import "https://kit.fontawesome.com/42d5adcbca.js";
</style>

<script>
import CryptrSpa from "@cryptr/cryptr-spa-js";
var firstIdp = null;
var adfsIdp = null;
var idpIds = null;
var idpByProvider = null;
var magicLinkActivated = null;

var firstDomain = '3-belges';
var secondDomain = 'tikcat';
if (process.env.VUE_APP_CRYPTR_IDP_IDS) {
  idpIds = process.env.VUE_APP_CRYPTR_IDP_IDS.split(",");
  firstIdp = idpIds[0];
  adfsIdp = idpIds[idpIds.length - 1];
} else {
  console.warn("no VUE_APP_CRYPTR_IDP_IDS");
}
if (process.env.VUE_APP_CRYPTR_IDPS_BY_PROVIDER) {
  var items = process.env.VUE_APP_CRYPTR_IDPS_BY_PROVIDER.split(",");
  idpByProvider = items.map((item) => {
    var parts = item.split(":");
    return {
      provider: parts[0],
      idpId: parts[1],
    };
  });
}

if (process.env.VUE_APP_CRYPTR_MAGIC_LINK) {
  magicLinkActivated = process.env.VUE_APP_CRYPTR_MAGIC_LINK === "true";
}

const config = {
  tenant_domain: process.env.VUE_APP_CRYPTR_TENANT_DOMAIN,
  client_id: process.env.VUE_APP_CRYPTR_CLIENT_ID,
  audience: process.env.VUE_APP_CRYPTR_AUDIENCE,
  default_redirect_uri: process.env.VUE_APP_CRYPTR_REDIRECT_URI,
  cryptr_base_url: process.env.VUE_APP_CRYPTR_BASE_URL,
  default_locale: process.env.VUE_APP_CRYPTR_DEFAULT_LOCALE,
  telemetry: process.env.VUE_APP_CRYPTR_TELEMETRY === "true",
  dedicated_server: process.env.VUE_APP_CRYPTR_DEDICATED_SERVER === "true",
  fixed_pkce: process.env.VUE_APP_CRYPTR_FIXED_PKCE === "true",
};

export default {
  // eslint-disable-next-line @typescript-eslint/explicit-module-boundary-types
  data() {
    return {
      loading: false,
      isAuthenticated: false,
      user: {},
      cryptrClient: null,
      error: null,
      idpIds: [],
      firstIdp: null,
      adfsIdp: null,
      idpByProvider: null,
      magicLinkActivated: magicLinkActivated,
    };
  },
  // eslint-disable-next-line @typescript-eslint/explicit-module-boundary-types
  async created() {
    this.firstIdp = firstIdp;
    this.idpIds = idpIds;
    this.adfsIdp = adfsIdp;
    this.idpByProvider = idpByProvider;
    this.magicLinkActivated = magicLinkActivated;
    this.firstDomain = firstDomain;
    this.secondDomain = secondDomain;
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
      console.error("error in App.vue Auth process");
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
