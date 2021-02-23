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
      curScene: this.recievedScene,
    };
  },
  props: {
    userAddress: String,
    tokenInfo: Object,
    recievedScene: String,
  },
  watch: {
    recievedScene: function (newscene, oldscene) {
      this.setScene(newscene);
    },
  },
  mounted() {
    this.width = window.innerWidth;
    this.height = window.innerHeight;
    //this.setScene("idle");
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
      this.bgCol = new THREE.Color("rgb(31, 33, 33)");
      this.scene.background = this.bgCol;
      {
        const near = 0;
        this.far = 6;
        this.scene.fog = new THREE.Fog(this.bgCol, near, this.far);
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

      // const geomSphere = new THREE.SphereGeometry(1, 12, 7);
      // const matSphere = new THREE.MeshBasicMaterial({
      //   color: 0xffffff,
      //   wireframe: true,
      //   opacity: 1,
      // });
      // const sphere = new THREE.Mesh(geomSphere, matSphere);
      // sphere.position.y = 1;
      // sphere.position.z = 1;
      // this.scene.add(sphere);

      const loader = new GLTFLoader();
      const ethurl = "./ethlogo.gltf";
      const coinurl = "./coin1.gltf";

      this.introBgObjects = [];
      console.log(this.introBgObjects);

      loader.load(ethurl, (gltf) => {
        let logo = this.cloneGltf(gltf);
        const root = logo.scene;
        this.introEthLogo = root;
        this.scene.add(this.introEthLogo);
        console.log("added eth to scene");

        // // Add eth logos to bg
        for (let i = 0; i < 20; i++) {
          // Apply new material to texture
          let model = this.cloneGltf(gltf).scene;
          let newMaterial = new THREE.MeshStandardMaterial({
            color: "rgb(31, 33, 33)",
            roughness: 0,
          });
          model.traverse((o) => {
            if (o.isMesh) o.material = newMaterial;
          });
          this.addIntroBgObject(model);
        }

        // from root and below
        const box = new THREE.Box3().setFromObject(root);
        const boxSize = box.getSize(new THREE.Vector3()).length();
        const boxCenter = box.getCenter(new THREE.Vector3());

        var maxAxis = Math.max(boxSize.x, boxSize.y, boxSize.z);

        this.introEthLogo.scale.multiplyScalar((1 / boxSize) * 2);
        this.introEthLogo.rotation.x += 0.36;
        this.introEthLogo.position.z = 0.6;
        this.ethLogoLoaded = true;
        // console.log(gltf);

        // set the camera to frame the box
        //frameArea(boxSize * 0.5, boxSize, boxCenter, this.camera);
      });

      const hemiLight = new THREE.HemisphereLight(0xffffff, 0x444444);
      hemiLight.position.set(0, 0, 0);
      this.scene.add(hemiLight);

      const dirLight = new THREE.DirectionalLight(0xffffff);
      dirLight.position.set(-3, 10, -10);
      dirLight.castShadow = true;
      dirLight.shadow.camera.top = 2;
      dirLight.shadow.camera.bottom = -2;
      dirLight.shadow.camera.left = -2;
      dirLight.shadow.camera.right = 2;
      dirLight.shadow.camera.near = 0.1;
      dirLight.shadow.camera.far = 40;

      //this.scene.add(dirLight);
      this.clock = new THREE.Clock();
      this.time = 0;
      this.delta = 0;

      const size = 100;
      const divisions = 90;

      const gridHelper = new THREE.GridHelper(size, divisions);
      //gridHelper.rotation.x += 0.2;
      gridHelper.position.y = 0;
      this.scene.add(gridHelper);

      this.animate();
    },

    handleResize() {
      this.renderer.setSize(window.innerWidth, window.innerHeight);
      this.camera.aspect = window.innerWidth / window.innerHeight;
      this.camera.updateProjectionMatrix();
      // console.log("canvas should've resized");
    },

    setScene(scene) {
      this.curScene = scene;
    },

    addIntroBgObject(obj) {
      // Add floating BG objects in intro scene
      let randx = Math.random() * 10 + 3;
      let randy = Math.random() * 5;
      let randz = Math.random() * 15;
      obj.position.x = Math.random() >= 0.5 ? 0 - randx : 0 + randx;
      obj.position.y = Math.random() >= 0.5 ? 0 - randy : 0 + randy;
      obj.position.z = -randz - 2;
      obj.vertMoveMult = Math.random() * 0.01;
      obj.vertDir = Math.random() >= 0.5 ? -1 : 1;
      this.introBgObjects.push(obj);
      this.scene.add(obj);

      const box = new THREE.Box3().setFromObject(obj);
      const boxSize = box.getSize(new THREE.Vector3()).length();
      const boxCenter = box.getCenter(new THREE.Vector3());

      var maxAxis = Math.max(boxSize.x, boxSize.y, boxSize.z);

      obj.scale.multiplyScalar((1 / boxSize) * Math.random() * 7);
      //obj.rotation.x += 0.36;
    },

    cloneGltf(gltf) {
      const clone = {
        animations: gltf.animations,
        scene: gltf.scene.clone(true),
      };

      const skinnedMeshes = {};

      gltf.scene.traverse((node) => {
        if (node.isSkinnedMesh) {
          skinnedMeshes[node.name] = node;
        }
      });

      const cloneBones = {};
      const cloneSkinnedMeshes = {};

      clone.scene.traverse((node) => {
        if (node.isBone) {
          cloneBones[node.name] = node;
        }

        if (node.isSkinnedMesh) {
          cloneSkinnedMeshes[node.name] = node;
        }
      });

      for (let name in skinnedMeshes) {
        const skinnedMesh = skinnedMeshes[name];
        const skeleton = skinnedMesh.skeleton;
        const cloneSkinnedMesh = cloneSkinnedMeshes[name];

        const orderedCloneBones = [];

        for (let i = 0; i < skeleton.bones.length; ++i) {
          const cloneBone = cloneBones[skeleton.bones[i].name];
          orderedCloneBones.push(cloneBone);
        }

        cloneSkinnedMesh.bind(
          new Skeleton(orderedCloneBones, skeleton.boneInverses),
          cloneSkinnedMesh.matrixWorld
        );
      }

      return clone;
    },

    updateIdle() {
      //console.log(this.curScene);
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
        if (this.far < 40) {
          this.far += 0.6;
          //console.log(this.curScene);
          this.scene.fog = new THREE.Fog(this.bgCol, 0, this.far);
        }
        // Animate background objects
        this.introBgObjects.forEach(function (obj) {
          obj.position.y += obj.vertMoveMult * obj.vertDir;
        });
      }

      this.introEthLogo.position.y = 0.2 * Math.abs(Math.sin(this.time) * 0.6);
    },

    updateIdleFadeout() {
      //console.log(this.curScene);
      let mult = 15;
      this.introEthLogo.rotation.y += 0.01 * mult;
      this.introEthLogo.position.y = 0.2 * Math.abs(Math.sin(this.time) * 0.6);
      // Animate background objects
      this.introBgObjects.forEach(function (obj) {
        obj.position.y += obj.vertMoveMult * obj.vertDir;
      });
      if (this.far > 0.6) {
        this.far -= 0.6;
      } else {
        this.far = 0;
        setTimeout(() => {
          this.setScene("main");
        }, 3000);
      }
      //this.far = this.far > 0 ? this.far - 1 : 0;
      this.scene.fog = new THREE.Fog(this.bgCol, 0, this.far);
    },

    sceneSpecificAnim() {
      if (this.curScene == "idle" && this.ethLogoLoaded) {
        this.updateIdle();
      } else if (this.curScene == "idleFadeout" && this.ethLogoLoaded) {
        this.updateIdleFadeout();
      }
    },

    animate() {
      // Animate specific scene
      this.sceneSpecificAnim();
      this.renderer.render(this.scene, this.camera);
      requestAnimationFrame(this.animate);
      // console.log(this.recievedScene, this.curScene);
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
