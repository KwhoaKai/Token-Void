<template>
  <transition name="fade">
    <div id="onboarding" v-show="show">
      <Greeting
        :loading="loadingWallet"
        :message="greetingMessage"
        :title="title"
      />
      <SignInMM
        v-on:set-info="setUserInfo"
        v-on:parsing-wallet="parsingWallet"
      />
    </div>
  </transition>
</template>
<script>
import Greeting from "./Greeting.vue";
import SignInMM from "./SignInMM.vue";

export default {
  name: "Onboard",
  data() {
    return {
      show: true,
      userAddress: null,
      tokenInfo: null,
      loadingWallet: false,
      title: "Welcome home, Crypto Cowboy.",
      greetingMessage: "Sign in with your MetaMask wallet so we can begin.",
    };
  },
  props: {
    connectedMM: Boolean,
    loading: Boolean,
  },
  methods: {
    setUserInfo(info) {
      this.userAddress = info.address;
      this.tokenInfo = info.tokenInfo;
      console.log("In onboard, tokenInfo: ", info);
      this.$emit("set-onboard-info", info);
      this.greetingMessage = "Success!";
      this.title = "Welcome.";
      setTimeout(() => {
        this.greetingMessage = "Let's begin.";
        setTimeout(() => {
          this.show = false;
          this.$emit("onboard-fadeout");
        }, 2000);
      }, 2000);
    },
    parsingWallet() {
      this.loadingWallet = true;
      this.greetingMessage = "Wallet connected, parsing information...";
    },
  },
  components: {
    Greeting,
    SignInMM,
  },
};
</script>
<style scoped>
h2 {
  font-size: 40px;
  color: rgb(36, 36, 36);
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

#onboarding {
  position: absolute;
  left: 0;
  right: 0;
  transform: translateY(23vh);
  margin-left: auto;
  margin-right: auto;
}
</style>