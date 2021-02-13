<template>
  <div id="greetDiv">
    <h2>{{ title }}</h2>
    <h3>{{ message }}</h3>
  </div>
</template>
<script>
module.exports = {
  name: "Greeting",
  props: {
    connectedMM: Boolean,
    loading: Boolean,
    message: String,
    title: String,
  },
  methods: {
    loadingWallet() {
      this.message = "Loading MetaMask wallet info";
    },
  },
};
</script>
<style scoped>
#greetDiv {
  /* font-family: "Roboto"; */
}

h2 {
  font-size: 60px;
  color: rgb(255, 255, 255);
}

h3 {
  color: rgb(255, 255, 255);
}
</style>