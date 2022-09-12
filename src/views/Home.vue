<template>
  <div class="home container">
    <img
      src="@/assets/logo_cryptr_horizontal.png"
      height="80"
      style="margin-bottom: 32px"
    />
    <HelloWorld msg="Welcome to Cryptr Vue Sample App" />
    <hr />
    <div>
      <div class="alert alert-danger" v-if="error">
        <span>{{ error }}</span>
      </div>
      <div class="alert alert-info" v-if="!loading && isAuthenticated">
        <span>Seems all good</span>
      </div>
      <span v-if="loading">Loading ..</span>
      <div class="pricing-header px-3 py-3 pt-md-5 pb-md-4 mx-auto text-center">
        <h1 class="display-4">Development mode</h1>
        <p class="lead">
          This local static site use our cryptr API on our test environment.
        </p>
        <button
          v-if="isAuthenticated"
          type="button"
          class="btn btn-danger mx-3"
          @click="cryptrClient.logOut()"
        >
          Logout
        </button>
        <router-link to="/private" class="btn btn-primary">Private</router-link>
      </div>
      <div class="card mb-4 box-shadow" v-if="!loading && idpByProvider">
        <div class="card-header">
          <h4 class="my-0 font-weight-normal">SSO Catalog</h4>
          <p>
            Here are some direct SSO buttons for some of our supported providers
          </p>
        </div>
        <div class="card-body">
          <button
            v-for="item in idpByProvider"
            v-bind:key="item.idpId"
            href="#"
            class="btn btn-outline-info mx-3"
            @click="cryptrClient.signInWithSSO(item.idpId)"
          >
            {{ item.provider }}
          </button>
        </div>
      </div>
      <div class="card mb-4 box-shadow" v-if="!loading">
        <div class="card-header">
          <h4 class="my-0 font-weight-normal">Gateway</h4>
          <p>You don't known what's user's IDP? use our gateway</p>
        </div>
        <div class="card-body">
          <button
            type="button"
            @click="cryptrClient.signInWithSSOGateway()"
            class="btn btn-success mx-3 btn-sm"
            href="#"
          >
            Global Gateway
          </button>
          <button
            type="button"
            @click="cryptrClient.signInWithSSOGateway(firstIdp)"
            class="btn btn-success mx-3 btn-sm"
            href="#"
          >
            Gateway with one IDP
          </button>
          <button
            type="button"
            @click="cryptrClient.signInWithSSOGateway(idpIds)"
            class="btn btn-success mx-3 btn-sm"
            href="#"
          >
            Gateway with multiple idps
          </button>
        </div>
      </div>
      <!-- REQUIRES the SAME CLIENT_ID FOR MASTER TENANT -->
      <div class="card mb-4 box-shadow" v-if="!loading && magicLinkActivated">
        <div class="card-header">
          <h4 class="my-0 font-weight-normal">Magic Link</h4>
          <p>Here is also a test of magic link</p>
        </div>
        <div class="card-body">
          <button
            href="#"
            class="btn btn-outline-primary mx-3"
            @click="cryptrClient.signInWithRedirect()"
          >
            Magic Link Login
          </button>
        </div>
      </div>
      <div v-if="user.email && !loading" class="card box-shadow">
        <div class="card-header">
          <h4>User</h4>
        </div>
        <div class="card-body">
          <p v-if="user.ips">
            You connected through {{ user.ips }} provider (using '{{
              user.sci
            }}' idp)
          </p>
          <ul>
            <li>Email: {{ user.email }}</li>
            <li>User ID: {{ user.sub }}</li>
            <li>Organization Domain: {{ user.tnt }}</li>
          </ul>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from "vue-property-decorator";
import HelloWorld from "@/components/HelloWorld.vue";

@Component({
  components: {
    HelloWorld,
  },
})
export default class Home extends Vue {
  @Prop() private isAuthenticated!: boolean;
  @Prop() private loading!: boolean;
  @Prop() private cryptrClient!: unknown;
  @Prop() private user!: unknown;
  @Prop() private error!: unknown;
  @Prop() private firstIdp!: string | null;
  @Prop() private adfsIdp!: string | null;
  @Prop() private idpIds!: unknown | null;
  @Prop() private idpByProvider!: unknown | null;
  @Prop() private magicLinkActivated!: unknown | null;
}
</script>
