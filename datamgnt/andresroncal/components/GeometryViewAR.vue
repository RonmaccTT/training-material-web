<template>
  <div id="viewport">
    <!-- To this element we will append our 3d scene. -->
    <div id="threejs-container"></div>
</div>
</template>

<script setup>
// Imports;
import { onMounted, onUpdated, watch } from 'vue'
import * as THREE from "three"
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls"
import { TransformControls } from 'three/addons/controls/TransformControls.js';
import { Rhino3dmLoader } from "three/addons/loaders/3DMLoader.js"
import { runCompute } from "@/scripts/compute.js"
import { loadRhino } from "@/scripts/compute.js";

const loader = new Rhino3dmLoader()
loader.setLibraryPath('https://cdn.jsdelivr.net/npm/rhino3dm@8.0.0-beta2/')


const props = defineProps(["data", "path", "runCompute"]);
const emits = defineEmits(["updateMetadata"]);  


watch(() => props.runCompute, (newValue) => {
  if (newValue) {
    compute();
  }
})

// Three js objects
let renderer, camera, scene,  controls, container
let points = []

function init() {
  // https://threejs.org/docs/#api/en/renderers/WebGLRenderer
  // This object will render our scene
  THREE.Object3D.DEFAULT_UP.set(0, 0, 1);

  // Create a scene and a camera
  scene = new THREE.Scene()
  scene.background = new THREE.Color(1, 1, 1)
  camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)
  camera.position.set(350, 450, 90)

  // Create the renderer and add it to the HTML
  renderer = new THREE.WebGLRenderer({ antialias: true })
  renderer.shadowMap.enabled = true
  renderer.setSize(window.innerWidth, window.innerHeight)
  renderer.setPixelRatio(window.devicePixelRatio)  

  container = document.getElementById("threejs-container")

  // We are taking element defined in <template> and appending our render to it. 
  container.appendChild(renderer.domElement)

  //Set default three.js up
  // THREE.Object3D.DEFAULT_UP = new THREE.Vector3(1, 0, 0);

  // orbit controls
  controls = new OrbitControls(camera, renderer.domElement)

  // add some ambient light here
  const ambientlight = new THREE.AmbientLight(0xffffff, 1)
  ambientlight.position.set(0, 0, 300)
  scene.add(ambientlight)

  const directionalLight = new THREE.DirectionalLight(0xffffff, 1)
  directionalLight.position.set(0, 1, 30)
  scene.add(directionalLight)
  // add fun shape
  animate()
}

// // this function runs Compute
async function compute() {
  console.log("Runnning compute... \ndata sent: ", props.data)
  const doc = await runCompute(props.data, props.path)

  if (doc.metadata) {
    emits("updateMetadata", doc.metadata);
  }

  // clear objects from scene
  scene.children.forEach(child => {
    if (!child.isLight) {
      scene.remove(child)
    }
  });

  // scene.traverse((child) => {
  //   if (!child.isLight) {
  //     scene.remove(child)
  //   }
  // })


  const buffer = new Uint8Array(doc.toByteArray()).buffer;
  loader.parse(buffer, function (object) {
    ///////////////////////////////////////////////////////////////////////
    // add object graph from rhino model to three.js scene
    object.traverse((child) => {

      const mat = new THREE.MeshLambertMaterial()
      child.material = mat
      console.log(child)

      if (child.userData.hasOwnProperty("attributes") && child.userData.attributes.hasOwnProperty("userStrings")) {

        console.log("Key-1: ", child.userData.attributes.userStrings[4][0])
        console.log("Value-1: ", child.userData.attributes.userStrings[4][1])

        // // Check if the value at index 5 is defined before accessing it
        // if (child.userData.attributes.userStrings && child.userData.attributes.userStrings[5] && child.userData.attributes.userStrings[5][0] !== undefined && child.userData.attributes.userStrings[5][1] !== undefined) {
        //     console.log("Key-2: ", child.userData.attributes.userStrings[5][0]);
        //     console.log("Value-2: ", child.userData.attributes.userStrings[5][1]);
        // } else {
        //     // If value is undefined or the array itself is undefined, remove it from the array
        //     if (child.userData.attributes.userStrings) {
        //         child.userData.attributes.userStrings.splice(5, 1);
        //     }
        // }
        // get color from userStrings
        const colorData = child.userData.attributes.userStrings[4]
        const col = colorData[1]

        if (child.isMesh) {
          //convert color from userstring to THREE color and assign it
          const threeColor = new THREE.Color("rgb(" + col + ")")
          const mat = new THREE.MeshLambertMaterial({ color: threeColor })
          child.material = mat
        }
      }
      console.log()
    })
    scene.add(object)
    // zoom to extents
    for (let child of scene.children) {
        if (child.type == "Object3D") {
          // zoom to extents
          zoomCameraToSelection(camera, controls, child.children)

        }
      }
    console.log("Compute done")
  });
}
// for controls update
function animate() {
  // https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame
  // native Javascript function that tells your browser you are animating!
  requestAnimationFrame(animate);
  controls.update();
  renderer.render(scene, camera);
}

// This will be run whenever the window is resized
window.addEventListener("resize", onWindowResize);
function onWindowResize() {

  const height = container.offsetHeight
  const width = container.offsetWidth

  camera.aspect = width/ height
  camera.updateProjectionMatrix();

  renderer.setSize(width, height)
}

/**
 * Helper function that behaves like rhino's "zoom to selection", but for three.js!
 */
function zoomCameraToSelection(camera, controls, selection, fitOffset = 1.1) {

  const box = new THREE.Box3();

  for (const object of selection) {
    if (object.isLight) continue
    box.expandByObject(object);
  }

  const size = box.getSize(new THREE.Vector3());
  const center = box.getCenter(new THREE.Vector3());

  const maxSize = Math.max(size.x, size.y, size.z);
  const fitHeightDistance = maxSize / (2 * Math.atan(Math.PI * camera.fov / 360));
  const fitWidthDistance = fitHeightDistance / camera.aspect;
  const distance = fitOffset * Math.max(fitHeightDistance, fitWidthDistance);

  const direction = controls.target.clone()
    .sub(camera.position)
    .normalize()
    .multiplyScalar(distance);
  controls.maxDistance = distance * 10;
  controls.target.copy(center);

  camera.near = distance / 100;
  camera.far = distance * 100;
  camera.updateProjectionMatrix();
  camera.position.copy(controls.target).sub(direction);

  controls.update();
}
// This will be run whenever this component is instantiated
onMounted(async() => {
  init()
  await loadRhino()
  // visualizeGumballPoints()
  // compute();
})
// //onUpdated is called when an input prop is changed
// // this runs compute whenever the input data changes
// onUpdated(() => {
//   compute();
// })

</script>

<style scoped>
#viewport {

  height: 100%;
  width: 100%;
  min-width: 200px;
  position:inherit;
}

#threejs-container {
  height: 100%;
  width: 100%;
  min-width: 200px;
  position:inherit;
}
</style>