<template>
  <div class="home container">
    <img
      src="@/assets/logo_cryptr_horizontal.png"
      height="80"
      style="margin-bottom: 32px"
    />
    <HelloWorld msg="Welcome to Cryptr Vue Sample App" />
    <hr/>
    <div>
      <span>{{ error }}</span>
      <span v-if="loading">Loading ..</span>
      <div class="pricing-header px-3 py-3 pt-md-5 pb-md-4 mx-auto text-center">
        <h1 class="display-4">Development mode</h1>
        <p class="lead">
          This local static site use our cryptr API on our test environment.
        </p>
        <button
          href="#"
          class="btn btn-secondary mx-3"
          v-if="!loading && !isAuthenticated && firstIdp"
          @click="cryptrClient.signInWithSSO(firstIdp)"
        >
          Signin with SSO
        </button>
        <button
          href="#"
          class="btn btn-secondary mx-3"
          v-if="!loading && !isAuthenticated && adfsIdp"
          @click="cryptrClient.signInWithSSO(adfsIdp)"
        >
          Signin with ADFS
        </button>
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
      <div class="card mb-4 box-shadow" v-if="!loading">
        <div class="card-header">
          <h4 class="my-0 font-weight-normal">Gateway</h4>
          <p>You don't known what's user's IDP? use our gateway</p>
        </div>
        <div class="card-body">
          <button
            type="button"
            @click="cryptrClient.signInWithSSOGateway()"
            class="btn btn-info mx-3 btn-sm"
            href="#"
          >
            Global Gateway
          </button>
          <button
            type="button"
            @click="cryptrClient.signInWithSSOGateway(firstIdp)"
            class="btn btn-info mx-3 btn-sm"
            href="#"
          >
            Gateway with one IDP
          </button>
          <button
            type="button"
            @click="cryptrClient.signInWithSSOGateway(idpIds)"
            class="btn btn-info mx-3 btn-sm"
            href="#"
          >
            Gateway with multiple idps
          </button>
        </div>
      </div>
      <div v-if="user && !loading" class="card box-shadow">
        <div class="card-header">
          <h4>User</h4>
        </div>
        <div class="card-body">
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
  @Prop() private cryptrClient!: boolean;
  @Prop() private user!: unknown;
  @Prop() private error!: unknown;
  @Prop() private firstIdp!: string | null;
  @Prop() private adfsIdp!: string | null;
  @Prop() private idpIds!: unknown | null;

}
</script>
