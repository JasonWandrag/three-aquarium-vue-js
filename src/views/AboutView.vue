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
  SphereGeometry,
  MeshBasicMaterial,
} from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import * as CANNON from "cannon-es";
import { onMounted, onUnmounted, ref } from "vue";

const canvas = ref(null);
const timestep = 1 / 20;

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
  gravity: new CANNON.Vec3(0, -20, 0),
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
const physicsFloorMat = new CANNON.Material()
const physicsFloor = new CANNON.Body({
  shape: new CANNON.Plane(),
  type: CANNON.Body.STATIC,
  material: physicsFloorMat
});
world.addBody(physicsFloor);
physicsFloor.quaternion.setFromEuler(-Math.PI / 2, 0, 0);

// Add ball to scene
const ballGeo = new SphereGeometry(2);
const ballMat = new MeshBasicMaterial({
  color: 0x000000,
  wireframe: true,
});
const ball = new Mesh(ballGeo, ballMat);
scene.add(ball);
const ballBodyMat = new CANNON.Material()
const ballBody = new CANNON.Body({
  mass: 100,
  shape: new CANNON.Sphere(2),
  position: new CANNON.Vec3(1, 20, 0),
  material: ballBodyMat
});
ballBody.linearDamping = 0.01;
world.addBody(ballBody);

// Setup ball bounce
const ballBounce = new CANNON.ContactMaterial(physicsFloorMat, ballBodyMat, {restitution: 0.6})
world.addContactMaterial(ballBounce)

// Controls animation frame by frame
function animate() {
  world.step(timestep);
  floor.position.copy(physicsFloor.position);
  floor.quaternion.copy(physicsFloor.quaternion);

  ball.position.copy(ballBody.position);
  ball.quaternion.copy(ballBody.quaternion);
  renderer.render(scene, camera);
}

// Start animation loop
renderer.setAnimationLoop(animate);

// Initiate Canvas
onMounted(() => {
  if (canvas.value) {
    canvas.value.appendChild(renderer.domElement);
  }
});
// Cleanup on component unmount
onUnmounted(() => {
  // Stop animation loop
  renderer.setAnimationLoop(null);

  // Dispose of Three.js resources
  renderer.dispose();
  
  // Dispose of Cannon.js resources
  world.removeEventListener('postStep', animate);

  // Additional cleanup if needed
  // ...

  // Clear the canvas
  if (canvas.value) {
    canvas.value.innerHTML = '';
  }
});
</script>

<style></style>
