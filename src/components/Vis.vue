<template>
  <div id="container">
    <canvas id="canvas"></canvas>
  </div>
</template>
<script>
import * as THREE from "three";
import { AsciiEffect } from "three/examples/jsm/effects/AsciiEffect";

export default {
  name: "Vis",
  data() {
    return {
      width: null,
      height: null,
    };
  },
  props: {
    userAddress: String,
    tokenInfo: Object,
  },
  mounted() {
    this.width = window.innerWidth;
    this.height = window.innerHeight;
    this.initThree();
    console.log("mounted vis");

    window.addEventListener("resize", this.handleResize);
  },
  created() {},
  destroyed() {
    window.removeEventListener("resize", this.handleResize);
  },
  methods: {
    initThree() {
      const canvas = document.getElementById("canvas");
      let width = window.innerWidth;
      let height = window.innerHeight;
      canvas.width = this.width;
      canvas.height = this.height;
      this.scene = new THREE.Scene();
      this.scene.background = new THREE.Color("#fff6de");
      this.camera = new THREE.PerspectiveCamera(75, width / height, 0.1, 1000);
      this.renderer = new THREE.WebGLRenderer({
        canvas: canvas,
        antialias: true,
        alpha: false,
      });
      this.renderer.setSize(width, height);

      let geometry = new THREE.BoxGeometry(1, 1, 1);
      let material = new THREE.MeshBasicMaterial({ color: 0x0000ff });
      this.cube = new THREE.Mesh(geometry, material);
      this.scene.add(this.cube);

      this.camera.position.z = 5;
      this.animate();
    },

    handleResize() {
      this.width = window.innerWidth;
      this.height = window.innerHeight;
    },

    animate() {
      function resizeRendererToDisplaySize(renderer) {
        const canvas = renderer.domElement;
        const width = canvas.clientWidth;
        const height = canvas.clientHeight;
        const needResize = canvas.width !== width || canvas.height !== height;
        if (needResize) {
          console.log("resiszing");
          renderer.setSize(width, height, false);
        }
        return needResize;
      }
      if (resizeRendererToDisplaySize(this.renderer)) {
        const canvas = this.renderer.domElement;
        this.camera.aspect = canvas.clientWidth / canvas.clientHeight;
        this.camera.updateProjectionMatrix();
        this.$nextTick();
      }
      this.cube.rotation.x += 0.01;
      this.renderer.render(this.scene, this.camera);
      requestAnimationFrame(this.animate);
    },
  },
};
</script>
<style>
#Vis {
  font-family: Helvetica, Arial, sans-serif;
  text-align: center;
}

#canvas {
  width: 100vw;
  height: 100vh;
  display: block;
}
</style>
