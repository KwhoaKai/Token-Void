<template>
  <div id="app">
    <Onboard
      v-on:set-onboard-info="setUserInfo"
      v-on:onboard-fadeout="idleFadeout"
    />
    <Scene
      :user-address="userAddress"
      :token-info="tokenInfo"
      :recievedScene="sentScene"
    />
  </div>
</template>
<script>
import Scene from "./components/Scene.vue";
import Onboard from "./components/Onboard.vue";

export default {
  name: "App",
  data() {
    return {
      userAddress: null,
      tokenInfo: null,
      loadingWallet: false,
      sentScene: "idle",
    };
  },
  methods: {
    setUserInfo(info) {
      console.log("Updated at top level to: ", info.tokenInfo);
      this.userAddress = info.address;
      this.tokenInfo = info.tokenInfo;
    },
    idleFadeout(info) {
      console.log("Onboard over, fadeout idle scene");
      this.sentScene = "idleFadeout";
    },
  },
  components: {
    Scene,
    Onboard,
  },
};
</script>
<style>
#app {
  font-family: Helvetica, Arial, sans-serif;
  text-align: center;
  color: #2c3e50;
  width: 100%;
}

html,
body {
  margin: 0;
  width: 100%;
  height: 100%;
}
</style>
