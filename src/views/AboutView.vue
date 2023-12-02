<template>
  <div ref="canvas"></div>
</template>

<script setup>
import {
  WebGLRenderer,
  Scene,
  PerspectiveCamera,
  MeshStandardMaterial,
  Mesh,
  AmbientLight,
  PlaneGeometry,
  DirectionalLight,
  AnimationMixer,
  Clock,
  Box3,
  Vector3, // Import Vector3 directly
  Quaternion, // Import Quaternion directly
} from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { FBXLoader } from "three/examples/jsm/loaders/FBXLoader";
import * as CANNON from "cannon-es";
import { onMounted, ref } from "vue";


const canvas = ref(null);
const timestep = 1 / 60;

// Setup Renderer
const renderer = new WebGLRenderer({ antialias: true, alpha: true });
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.shadowMap.enabled = true;

// Setup Scene
const scene = new Scene();
// Setup Camera
const camera = new PerspectiveCamera(
  75,
  window.innerWidth / window.innerHeight,
  0.1,
  1000
);
camera.position.set(15, 15, 55);

// Setup Physics Engine
const world = new CANNON.World({
  gravity: new CANNON.Vec3(0, -9.81, 0),
});

// Setup view control
const orbitControls = new OrbitControls(camera, renderer.domElement);
orbitControls.update();

// Setup Lighting
const ambientLight = new AmbientLight(0x333333);
scene.add(ambientLight);
const directionalLight = new DirectionalLight(0xffffff, 10);
directionalLight.position.set(-10, 200, 10);
directionalLight.castShadow = true;
directionalLight.shadow.camera.bottom = -20;
directionalLight.shadow.camera.top = 20;
directionalLight.shadow.camera.left = 20;
directionalLight.shadow.camera.right = -20;
scene.add(directionalLight);

// Setup Floor
const floorGeo = new PlaneGeometry(1000, 1000);
const floorMat = new MeshStandardMaterial({ color: 0xb3bf });
const floor = new Mesh(floorGeo, floorMat);
floor.rotation.x = -0.5 * Math.PI;
floor.receiveShadow = true;
scene.add(floor);
const physicsFloor = new CANNON.Body({
  shape: new CANNON.Plane(),
  type: CANNON.Body.STATIC,
});
world.addBody(physicsFloor);
physicsFloor.quaternion.setFromEuler(-Math.PI / 2, 0, 0);

// Controls animation frame by frame
function animate() {
  world.step(timestep);
  floor.position.copy(physicsFloor.position);
  floor.quaternion.copy(physicsFloor.quaternion);
  // if (!loadedObjects.length) return;
  // loadedObjects.forEach((obj) => {
  //   obj.mixer.update(obj.clock.getDelta());
  //   obj.model.position.copy(obj.physicsBody.position);
  //   obj.model.quaternion.copy(obj.physicsBody.quaternion);
  // });
  // const delta = time.update().getDelta();
  // entityManager.update(delta);
  renderer.render(scene, camera);
}

renderer.setAnimationLoop(animate);


// Initiate Canvas
onMounted(() => {
  if (canvas.value) {
    canvas.value.appendChild(renderer.domElement);
  }
});
</script>

<style>
</style>
