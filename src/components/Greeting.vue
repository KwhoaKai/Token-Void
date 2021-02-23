<template>
  <div id="greetDiv">
    <div id="titleDiv"></div>
    <!-- <h2>{{ title }}</h2> -->
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
  watch: {
    title: function (newTitle, oldTitle) {
      const titleDiv = document.getElementById("titleDiv");
      while (titleDiv.firstChild) {
        titleDiv.removeChild(titleDiv.lastChild);
      }
      var text = new Blotter.Text(`${newTitle}`, {
        family: "'EB Garamond', serif",
        size: 80,
        fill: "#FFFFFF",
      });

      var material = new Blotter.LiquidDistortMaterial();

      // Play with the value for uSpeed. Lower values slow
      // down animation, while higher values speed it up. At
      // a speed of 0.0, animation is stopped entirely.
      material.uniforms.uSpeed.value = 0.25;
      material.uniforms.uVolatility.value = 0.015;

      // Try uncommenting the following line to play with
      // the "volatility" of the effect. Higher values here will
      // produce more dramatic changes in the appearance of your
      // text as it animates, but you will likely want to keep
      // the value below 1.0.
      //material.uniforms.uVolatility.value = 0.30;

      var blotter = new Blotter(material, {
        texts: text,
      });

      var elem = document.getElementById("titleDiv");
      console.log(elem);
      var scope = blotter.forText(text);

      scope.appendTo(elem);
    },
  },
  mounted() {
    var text = new Blotter.Text("Welcome home, Crypto Cowboy", {
      family: "'EB Garamond', serif",
      size: 80,
      fill: "#FFFFFF",
    });

    var material = new Blotter.LiquidDistortMaterial();

    // Play with the value for uSpeed. Lower values slow
    // down animation, while higher values speed it up. At
    // a speed of 0.0, animation is stopped entirely.
    material.uniforms.uSpeed.value = 0.25;
    material.uniforms.uVolatility.value = 0.015;

    // Try uncommenting the following line to play with
    // the "volatility" of the effect. Higher values here will
    // produce more dramatic changes in the appearance of your
    // text as it animates, but you will likely want to keep
    // the value below 1.0.
    //material.uniforms.uVolatility.value = 0.30;

    var blotter = new Blotter(material, {
      texts: text,
    });

    var elem = document.getElementById("titleDiv");
    var scope = blotter.forText(text);

    scope.appendTo(elem);
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
  font-family: "Nanum Myeongjo", serif;
  width: 100%;
  /* font-family: "Shippori Mincho", serif; */
}

#titleDiv {
  width: 100%;
}

canvas {
  width: 100%;
}

h2 {
  font-size: 60px;
  color: rgb(255, 255, 255);
}

h3 {
  color: rgb(255, 255, 255);
  font-family: "Arial";
}
</style>