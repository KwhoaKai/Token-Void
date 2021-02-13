<template>
  <div id="container">
    <canvas id="canvas"></canvas>
  </div>
</template>
<script>
import * as THREE from "three";
import { AsciiEffect } from "three/examples/jsm/effects/AsciiEffect";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";

export default {
  name: "Scene",
  data() {
    return {
      width: null,
      height: null,
      ethLogoLoaded: false,
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
      this.scene.background = new THREE.Color("rgb(36, 32, 32)");
      {
        const color = "rgb(36, 32, 32)"; // white
        const near = 0;
        this.far = 6;
        this.scene.fog = new THREE.Fog(color, near, this.far);
      }
      this.camera = new THREE.PerspectiveCamera(75, width / height, 0.1, 1000);
      this.camera.position.z = 4;
      this.camera.position.y = 1;
      this.renderer = new THREE.WebGLRenderer({
        canvas: canvas,
        antialias: true,
        alpha: false,
      });
      this.renderer.setSize(width, height);

      const loader = new GLTFLoader();
      const ethurl = "./ethlogo.gltf";

      loader.load(ethurl, (gltf) => {
        const root = gltf.scene;
        this.introEthLogo = root;
        this.scene.add(this.introEthLogo);
        console.log("added eth to secene");

        // from root and below
        const box = new THREE.Box3().setFromObject(root);
        const boxSize = box.getSize(new THREE.Vector3()).length();
        const boxCenter = box.getCenter(new THREE.Vector3());

        var maxAxis = Math.max(boxSize.x, boxSize.y, boxSize.z);

        this.introEthLogo.scale.multiplyScalar((1 / boxSize) * 2);
        this.introEthLogo.rotation.x += 0.36;
        this.ethLogoLoaded = true;
        console.log(gltf);

        // set the camera to frame the box
        //frameArea(boxSize * 0.5, boxSize, boxCenter, this.camera);
      });

      let geometry = new THREE.BoxGeometry(1, 1, 1);
      let material = new THREE.MeshBasicMaterial({ color: 0x0000ff });
      this.cube = new THREE.Mesh(geometry, material);
      //this.scene.add(this.cube);

      const hemiLight = new THREE.HemisphereLight(0xffffff, 0x444444);
      hemiLight.position.set(0, 20, 0);
      //scene.add(hemiLight);

      const dirLight = new THREE.DirectionalLight(0xffffff);
      dirLight.position.set(-3, 10, -10);
      dirLight.castShadow = true;
      dirLight.shadow.camera.top = 2;
      dirLight.shadow.camera.bottom = -2;
      dirLight.shadow.camera.left = -2;
      dirLight.shadow.camera.right = 2;
      dirLight.shadow.camera.near = 0.1;
      dirLight.shadow.camera.far = 40;

      this.scene.add(dirLight);
      this.clock = new THREE.Clock();
      this.time = 0;
      this.delta = 0;

      const size = 200;
      const divisions = 200;

      const gridHelper = new THREE.GridHelper(size, divisions);
      gridHelper.rotation.x += 0.2;
      this.scene.add(gridHelper);

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
      this.cube.rotation.y += 0.01;

      if (this.ethLogoLoaded) {
        this.delta = this.clock.getDelta();
        this.time += this.delta;

        let mult;
        if (!this.userAddress && !this.tokenInfo) {
          mult = 1;
        } else if (this.userAddress && !this.tokenInfo) {
          mult = 4;
        } else {
          mult = 15;
          this.introEthLogo.rotation.y += 0.01 * mult;
          if (this.far < 60) {
            this.far += 1;
          }
        }
        //console.log(mult);
        //console.log(this.far);

        this.introEthLogo.position.y =
          0.2 * Math.abs(Math.sin(this.time) * 0.6);
        const color = "rgb(36, 32, 32)"; // white

        this.scene.fog = new THREE.Fog(color, 0, this.far);
      }
      this.renderer.render(this.scene, this.camera);
      requestAnimationFrame(this.animate);
    },
  },
};
</script>
<style>
#canvas {
  width: 100vw;
  height: 100vh;
  display: block;
}
</style>
