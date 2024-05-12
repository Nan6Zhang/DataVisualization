<script>
    import { parse } from 'papaparse';
  
    const materialsCSV = `"MaterialKey","Material"
  "1","EVCB-001"
  "2","HB-001"
  "3","BC-001"
  "4","BMS-001"
  "5","C-001"
  "6","C-002"
  "7","CC-001"
  "8","CF-001"
  "9","CF-002"
  "10","ECB-001"
  "11","EHB-001"
  "12","F-001"
  "13","F-002"
  "14","F-003"
  "15","HE-001"
  "16","HE-002"
  "17","IM-001"
  "18","IM-002"
  "19","IM-003"
  "20","IN-001"
  "21","MS-001"
  "22","OEC-001"
  "23","OEC-002"
  "24","OEC-003"
  "25","OEC-004"
  "26","OEC-005"
  "27","R-001"
  "28","R-002"
  "29","R-003"
  "30","TS-001"
  "31","TS-002"
  "32","W-001"
  "33","W-002"`;
  
    const plantsCSV = `"PlantKey","Plant"
  "1","ANT1"
  "2","WRO1"
  "3","LYO1"
  "4","ANT2"
  "5","WRO2"
  "6","LYO2"
  "7","BIR2"
  "8","GOT2"`;
  
    const materialsData = parse(materialsCSV, { header: true }).data;
    const plantsData = parse(plantsCSV, { header: true }).data;
  
    const materialNames = materialsData.map(material => material.Material);
    const plantNames = plantsData.map(plant => plant.Plant);
  
    let selectedMaterialName = '';
    let selectedPlantName = '';
    let materialKey = '';
    let plantKey = '';
  
    function updateKeys() {
      const material = materialsData.find(item => item.Material === selectedMaterialName);
      const plant = plantsData.find(item => item.Plant === selectedPlantName);
      materialKey = material ? material.MaterialKey : '';
      plantKey = plant ? plant.PlantKey : '';
    }
  
    function handleMaterialNameChange(event) {
      selectedMaterialName = event.target.value;
      updateKeys();
    }
  
    function handlePlantNameChange(event) {
      selectedPlantName = event.target.value;
      updateKeys();
    }
  </script>
  
  <style>
    /* Add your styles here */
  </style>
  
<!-- 
<h1>Material and Plant Mapping</h1> -->
  
  <div>
    <label for="material-name">Select Material Name:</label>
    <select id="material-name" bind:value={selectedMaterialName} on:change={handleMaterialNameChange}>
      {#each materialNames as name}
        <option value={name}>{name}</option>
      {/each}
    </select>
  </div>
  
  <div>
    <label for="plant-name">Select Plant Name:</label>
    <select id="plant-name" bind:value={selectedPlantName} on:change={handlePlantNameChange}>
      {#each plantNames as name}
        <option value={name}>{name}</option>
      {/each}
    </select>
  </div>
  
  <div>
    <p>Selected Material Key: {materialKey}</p>
    <p>Selected Plant Key: {plantKey}</p>
  </div>
  
  