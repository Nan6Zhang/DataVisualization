<script>
    import { onMount } from 'svelte';
    import { csv } from 'd3-fetch';
    import { Chart, ArcElement, Tooltip, Legend } from 'chart.js';
  
    Chart.register(ArcElement, Tooltip, Legend);
  
    let ctx;
    let allData = [];
    let filteredData = [];
    let years = [];
    let months = [];
    let plantKeys = [];
    let selectedYear = null;
    let selectedMonth = null;
    let selectedPlantKey = null;
    let myChart = null; // Reference to Chart instance for updates
  
    onMount(async () => {
      allData = await csv('/data/Inventory.csv', d => ({
        MaterialKey: +d.MaterialKey,
        PlantKey: +d.PlantKey,
        SnapshotDate: new Date(d.SnapshotDate),
        GrossInventoryQuantity: +d.GrossInventoryQuantity,
        OnShelfInventoryQuantity: +d.OnShelfInventoryQuantity,
        InTransitQuantity: +d.InTransitQuantity
      }));
  
      years = Array.from(new Set(allData.map(d => d.SnapshotDate.getFullYear()))).sort();
      months = Array.from(new Set(allData.map(d => d.SnapshotDate.getMonth() + 1))).sort((a, b) => a - b);
      plantKeys = Array.from(new Set(allData.map(d => d.PlantKey))).sort((a, b) => a - b);
  
      updateData(); // Initial chart display
    });
  
    function updateData() {
      filteredData = allData.filter(d => 
        (selectedYear ? d.SnapshotDate.getFullYear() === selectedYear : true) &&
        (selectedMonth ? d.SnapshotDate.getMonth() + 1 === selectedMonth : true) &&
        (selectedPlantKey ? d.PlantKey === selectedPlantKey : true)
      );
      updateChart();
    }
  
    function updateChart() {
      const colors = ['#FF6384', '#36A2EB', '#FFCE56', '#4BC0C0', '#9966FF', '#FF9F40'];
      const detailColors = ['#C9CBFF', '#FFC9DE', '#7ED321', '#9013FE', '#FFD500', '#50E3C2']; // additional colors

      const materialKeys = [...new Set(filteredData.map(item => item.MaterialKey))];
      const aggregatedData = materialKeys.map((key, index) => {
        const relevantItems = filteredData.filter(item => item.MaterialKey === key);
        const totalGross = relevantItems.reduce((acc, item) => acc + item.GrossInventoryQuantity, 0);
        const totalOnShelf = relevantItems.reduce((acc, item) => acc + item.OnShelfInventoryQuantity, 0);
        const totalInTransit = relevantItems.reduce((acc, item) => acc + item.InTransitQuantity, 0);
        return {
          MaterialKey: key,
          Gross: totalGross,
          OnShelf: totalOnShelf,
          InTransit: totalInTransit,
          color: colors[index % colors.length]
        };
      });

      if (myChart) {
        myChart.destroy();
      }

      const outerData = aggregatedData.map(item => ({
        value: item.Gross,
        color: item.color
      }));

      const innerData = aggregatedData.flatMap((item, index) => ([
        { value: item.OnShelf, color: detailColors[(index * 2) % detailColors.length], label: `Material ${item.MaterialKey} On-Shelf`, total: item.Gross },
        { value: item.InTransit, color: detailColors[(index * 2 + 1) % detailColors.length], label: `Material ${item.MaterialKey} In-Transit`, total: item.Gross } // use a different color from detailColors
      ]));

      myChart = new Chart(ctx, {
        type: 'pie',
        data: {
          datasets: [
            {
              label: 'Gross Inventory',
              data: outerData.map(item => item.value),
              backgroundColor: outerData.map(item => item.color),
              hoverOffset: 4
            },
            {
              label: 'Detailed Inventory',
              data: innerData.map(item => item.value),
              backgroundColor: innerData.map(item => item.color),
              hoverOffset: 4,
              borderWidth: 2
            }
          ]
        },
        options: {
          responsive: true,
          plugins: {
            tooltip: {
              callbacks: {
                label: function(tooltipItem) {
                  const dataset = tooltipItem.dataset;
                  const index = tooltipItem.dataIndex;
                  const total = dataset.data.reduce((acc, value) => acc + value, 0);
                  const percentage = ((dataset.data[index] / total) * 100).toFixed(2) + '%';
                  const label = dataset.label === 'Detailed Inventory' ? innerData[index].label : `Material ${aggregatedData[index].MaterialKey}`;
                  return `${label}: ${dataset.data[index]} (${percentage})`;
                }
              }
            },
            legend: {
              display: true,
              position: 'top',
              labels: {
                generateLabels: function(chart) {
                  const datasets = chart.data.datasets;
                  const labels = datasets.flatMap((dataset, i) => {
                    return dataset.data.map((data, index) => ({
                      text: i === 0 ? `Material ${aggregatedData[index].MaterialKey} Gross Inventory` : innerData[index].label,
                      fillStyle: dataset.backgroundColor[index],
                    }));
                  });
                  return labels;
                }
              }
            }
          }
        }
      });
    }
  
    function handleYearChange(event) {
      selectedYear = +event.target.value || null;
      updateData();
    }
  
    function handleMonthChange(event) {
      selectedMonth = +event.target.value || null;
      updateData();
    }
  
    function handlePlantKeyChange(event) {
      selectedPlantKey = +event.target.value || null;
      updateData();
    }
</script>
  
<style>
    .selector-bar {
      display: flex;
      justify-content: space-evenly;
      margin-bottom: 20px;
    }
  
    select {
      padding: 8px;
      margin-right: 10px;
      border-radius: 4px;
      border: 1px solid #ccc;
    }
  
    label {
      margin-right: 5px;
    }
</style>
  
<main>
    <div class="selector-bar">
      <div>
        <label for="year-select">Select Year:</label>
        <select id="year-select" on:change={handleYearChange}>
          <option value="">All Years</option>
          {#each years as year}
            <option value={year}>{year}</option>
          {/each}
        </select>
      </div>
  
      <div>
        <label for="month-select">Select Month:</label>
        <select id="month-select" on:change={handleMonthChange}>
          <option value="">All Months</option>
          {#each months as month}
            <option value={month}>{month}</option>
          {/each}
        </select>
      </div>
  
      <div>
        <label for="plant-key-select">Select Plant Key:</label>
        <select id="plant-key-select" on:change={handlePlantKeyChange}>
          <option value="">All Plant Keys</option>
          {#each plantKeys as key}
            <option value={key}>{key}</option>
          {/each}
        </select>
      </div>
    </div>
  
    <canvas bind:this={ctx}></canvas>
</main>
