<script setup>
import { ref, onBeforeMount, computed } from "vue"
import { loadRhino } from "@/scripts/compute.js"

// Import other Vue components in order to add them to a template.
// import Header from "./components//HeaderAR.vue"
import GeometryView from "./components/GeometryViewAR.vue"
// import SliderInput from "commonComponents/SliderInput.vue"
import SliderInput from "./components/SliderInput.vue"
// import ToggleInput from "commonComponents/ToggleInput.vue"
import ToggleInput from "./components/ToggleInput.vue"
import DropdownSelector from "./components/DropdownSelectorAR.vue"
import ComputeButton from "./components/ComputeButtonAR.vue"
import Upload3dm from "commonComponents/Upload3dm.vue"
import def from './assets/geometry_copies.gh' //import Grasshopper definition for assets

let sliderName = ref("Count") //must match the Input name in your GH definition!
let sliderValue = ref(10) //default slider value

// City District One
let dropdownZones = ref("City District One")
console.log(dropdownZones )
let dropdownIndex = ref(0)
const dropdownOptions = [
  { label: "Fort Greene", value: 0 },
  { label: "Williamsburg", value: 1 },
  { label: "Bedstuy", value: 2 },
  { label: "Midwood", value: 3 }
];

// // City Amenities One
let dropdownZones2 = ref("City Amenities One")
let dropdownIndex2 = ref(0)
const dropdownOptions2 = [
  { label: "Parks", value: 0 },
  { label: "Education", value: 1},
  { label: "Hospitality", value: 2 },
  { label: "Transportation", value: 3 }
];

// City District Two
let dropdownZones3 = ref("City District Two")
console.log(dropdownZones3 )
let dropdownIndex3 = ref(0)
const dropdownOptions3 = [
  { label: "Fort Greene", value: 0 },
  { label: "Williamsburg", value: 1 },
  { label: "Bedstuy", value: 2 },
  { label: "Midwood", value: 3 }
];

// // City Amenities Two
let dropdownZones4 = ref("City Amenities Two")
let dropdownIndex4 = ref(0)
const dropdownOptions4 = [
  { label: "Parks", value: 0 },
  { label: "Education", value: 1},
  { label: "Hospitality", value: 2 },
  { label: "Transportation", value: 3 }
];

// let encodedFile = ref(null);
let isButtonDisabled = ref(false)
///.............................................
let path = def //path to the Grasshopper definition
let data = ref({})
let metadata = ref([])
let compute = ref(false)

function updateValue(newValue, parameterName) {
  if (parameterName === sliderName.value) {
    sliderValue.value = newValue
  }
  // else if (parameterName === toggleName.value) {
  //   toggleValue.value = newValue
  // }
  else if (parameterName === dropdownZones.value) {
    dropdownIndex.value = newValue
  }
  else if (parameterName === dropdownZones2.value) {
    dropdownIndex2.value = newValue
  }
  else if (parameterName === dropdownZones3.value) {
    dropdownIndex3.value = newValue
  }
  else if (parameterName === dropdownZones4.value) {
    dropdownIndex4.value = newValue
  }
  // console.log(parameterName + " : " + newValue)
}

// 3DM FILE CALL OUT HERE
// encodedFile.value = "./assets/Program massing.3dm"

// function update3dmData(newData) {
//   encodedFile.value = newData
//   isButtonDisabled.value = false
//   compute.value = true
// }

function receiveMetadata(newValue) {
  console.log(newValue)
  metadata.value = newValue
}
function runCompute(newVal) {
  compute.value = newVal
}
// a computed ref. Vue will keep track of this and update it
const computeData = computed(() => {
  data = {
    ["encodedFile"]: encodedFile.value,
    [sliderName.value]: Number(sliderValue.value),
    // [toggleName.value]: Boolean(toggleValue.value),
    [dropdownZones.value]: Number(dropdownIndex.value),
    [dropdownZones2.value]: Number(dropdownIndex2.value),
    [dropdownZones3.value]: Number(dropdownIndex3.value),
    [dropdownZones4.value]: Number(dropdownIndex4.value)
  };
  return data
})
</script>

<template>  
  <div id="sidebar" class="container">
    <img id="HLogo" class="logo-image" alt="Housing Pulse Logo" src="./assets/img/logoHousingPulse.png" />
    <!-- <h3>HousingPulse</h3> -->
    <div id="main-text-box">
      <div id="text" style="overflow-y: scroll; height:170px;">
        <p id="para"><strong>HousingPulse</strong> utilizes an internalized .3dm file containing four areas within Brooklyn, New York City. The model, developed with the Urbano plugin, is juxtaposed with amenities listed in a CSV file sourced from the website's repository. This CSV file contains records of New York City home sales data spanning from 2023 to 2024. Additionally, users have the option to download a CSV file encompassing sales data from Brooklyn compared to the data from OSM.
        <br><br>
        The ultimate goal is to provide user with an array of visual and numerical data, enabling them to identify areas that could benefit from further community interventions, such as the establishment of schools or housing facilities.
        </p>
        <a id="site-code" href="https://github.com/iaac-macad/bimsc24-final-project/tree/ronmaccms/final_project" style="color: #141414;" target="_blank">
          Site code
        </a>
      </div>    
      <br><br>
      <div id="dropm-w">
        <!-- DROP DOWN MENUS FOR THE CITIES -->
        <!-- CITY 1 -->
        <DropdownSelector :title="dropdownZones" :options="dropdownOptions" :val="dropdownIndex" @update="updateValue" />
        <DropdownSelector :title="dropdownZones2" :options="dropdownOptions2" :val="dropdownIndex2" @update="updateValue" />
        <h4 id="btn-txt">Compare data between these two cities</h4>
        <!-- CITY 2 -->
        <DropdownSelector :title="dropdownZones3" :options="dropdownOptions3" :val="dropdownIndex3" @update="updateValue" />
        <DropdownSelector :title="dropdownZones4" :options="dropdownOptions4" :val="dropdownIndex4" @update="updateValue" />
      </div>
      <br>
      <!-- COMPUTE BUTTON - REFRESH GEOMETRY -->
      <h4 id="btn-txt">Click to load the new selection</h4>
      <ComputeButton id="compute-btn" title="Compute" @click="runCompute" :isDisabled="isButtonDisabled" />
      <br>
      <!-- DISPLAY META DATA  -->
      <h4 id="btn-txt">Properties sales values &<br>area amenities</h4>
      <br>
      <p v-if="metadata && metadata.length > 0"> {{ metadata[0].name }} : {{ metadata[0].value }}</p>
    </div>
  </div>
  <div id="viewer" class="container">
      <GeometryView :data="computeData" :path="path" :runCompute="compute" @updateMetadata="receiveMetadata" />
  </div>
</template>
 
<style scoped>
.container{
background-color: rgb(59, 59, 59);
border-style: dotted;
border-width: 1px;
font-size: 12px; 
}

#content {
display: flex;
  height: calc(100vh - 100px);
  font-family: 'Franklin Gothic Medium', 'Arial Narrow', Arial, sans-serif;
  font-size: 14px; 
}

#text {
  background-color: white;
}

#btn-txt {
  font-size: x-small;
  text-align: center;
}

#para {
  font-size: small;
  padding-left: 20px;
  padding-right: 20px;
  text-align: justify;
  color: #141414;
  /* font-weight: bold; */
}

#main-text-box {
  margin-top: 30px;
  margin-left: 20px;
  margin-right: 20px;
}

#compute-btn {
  color: red;
}

#site-code {
  font-size: small;
  padding: 15px;
  text-align: justify;
  align-items: center;
  margin-left: 6.6rem;  
  color: #141414;
}

#HLogo {
  display: flex;
  align-items: center;
  margin-left: 6.7rem;
  margin-top: 10px;
  scale: 200%;
  padding: 30px
}

#top-bar2 {
  display: flex;
  align-items: center;
  justify-content: space-between;
  background-color: rgba(233, 233, 233, 0.7);
}

#top-bar2-title {
  display: flex;
  align-items: right;
}

#title-container2 {
  display: flex;
  align-items: center;
  color: rgb(0, 0, 0);
  margin-left: 2.5rem;
}

#sidebar {
  width: 350px;
  padding: 10px;
  font-size: 14px; 
  background-color: #2e4057;
  color: white;
}

#viewer {
  width: calc(100% - 200px);
  display: flex;
}

.modern-range {
  -webkit-appearance: none;
  width: 100%;
  background: linear-gradient(90deg, #ffffff, #ff0080);
  height: 17px;
  border-radius: 15px;
  margin: 10px 0px;
}

.modern-range::-webkit-slider-thumb {
  -webkit-appearance: none;
  height: 12px;
  width: 12px;
  border-radius: 12px;
  background-color: black;
  cursor: pointer;
}
</style>
