<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    <p>
      <span>
        For a complete guide see
        <a href="https://www.npmjs.com/package/@cryptr/cryptr-spa-js">
          @cryptr/cryptr-spa-js
        </a>
      </span>
    </p>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from "vue-property-decorator";

@Component
export default class HelloWorld extends Vue {
  @Prop() private msg!: string;
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
