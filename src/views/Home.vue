<template>
  <div class="home">
    <img alt="Cryptr logo" src="https://www.cryptr.co/" />
    <HelloWorld msg="Welcome to Cryptr Vue Sample App" />
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import HelloWorld from "@/components/HelloWorld.vue"; // @ is an alias to /src

@Component({
  components: {
    HelloWorld,
  },
})
export default class Home extends Vue {}
</script>
