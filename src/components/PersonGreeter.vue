<template>
  <div v-show="show">
    <h2>Hello there, Crypto Cowboy...</h2>
    <h3>{{ message }}</h3>
  </div>
</template>
<script>
module.exports = {
  name: "Greeting",
  data() {
    let message;
    if(this.loading && connectedMM) {
      message = "Connect your MetaMask wallet so we can begin your voyage.";
      this.show = false;
    } else if()
    return {
      message: message,
      show: true,
    };
  },
  props: {
    personName: String,
    connectedMM: Boolean,
    loading: Boolean,
  },
};
</script>
<style scoped>
h2 {
  font-size: 40px;
  color: rgb(36, 36, 36);
}
</style>