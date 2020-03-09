<template>
  <header class="header">
    <img src="../assets/images/twitter-poem-generator.png" alt="" />
    <h1>Twitter Poetry Generator</h1>
  </header>
</template>

<script lang="ts">
import { Component, Prop, Vue } from 'vue-property-decorator';

@Component
export default class Header extends Vue {}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss">
.header {
  padding: 2em 1em;

  background-color: $yellow;

  text-align: center;
  h1 {
    margin: 0;

    color: $violet-700;
    font-size: 2em;
  }
  img {
    display: block;
    margin: 0 auto 1em;
    height: auto;
    width: 10em;
    max-width: 80vw;
  }
}

</style>
