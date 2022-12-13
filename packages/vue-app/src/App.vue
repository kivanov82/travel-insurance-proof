<template>
  <div id="app">
    <img alt="Ethereum logo" src="./assets/stake-2022-1.png" width="400" height="400" />
    <HelloWorld msg="My Wallet"/>
  </div>
</template>

<script>
import HelloWorld from "./components/HelloWorld.vue";

export default {
  name: "EthereumApp",
  components: {
    HelloWorld,
  },
};
</script>

<style>
html, body {
  height: 100%;
  margin: 0;
  padding: 0;
  background-color: black;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: white;
  background-color: black;
  margin-top: 20px;
}
</style>
