<script>
    import { onMount } from 'svelte';
    import { Chart } from 'chart.js/auto';
    import { csvParse } from 'd3-dsv';
    import { createEventDispatcher } from 'svelte';

    const dispatch = createEventDispatcher();
    let selectedYear = '';
    let selectedMaterial = '';
    let selectedPlant = '';

    let chart;
    let chartData = [];

    // Load CSV data
    let data = [];

    onMount(async () => {
        const response = await fetch('/data/aggregated_inventory1.csv');
        const csvData = await response.text();
        data = csvParse(csvData);

        // Prepare initial chart data
        chartData = prepareChartData(data);
        renderChart(chartData);
    });

    function prepareChartData(data) {
        return data.map(entry => ({
            date: new Date(entry.SnapshotDate).toLocaleDateString(),
            year: new Date(entry.SnapshotDate).getFullYear(),
            material: entry.MaterialKey,
            plant: entry.PlantKey,
            grossInventoryQuantity: +entry.GrossInventoryQuantity,
            onShelfInventoryQuantity: +entry.OnShelfInventoryQuantity,
            inTransitQuantity: +entry.InTransitQuantity
        }));
    }

    function reduceData(data) {
        return data.reduce((acc, entry) => {
            let key = `${entry.date}`;
            if (!acc[key]) {
                acc[key] = {...entry, count: 1};
            } else {
                acc[key].grossInventoryQuantity += entry.grossInventoryQuantity;
                acc[key].onShelfInventoryQuantity += entry.onShelfInventoryQuantity;
                acc[key].inTransitQuantity += entry.inTransitQuantity;
                acc[key].count++;
            }
            return acc;
        }, {});
    }

    function filterData() {
        let filteredData = chartData;
        if (selectedYear && selectedYear !== "All Years") filteredData = filteredData.filter(entry => entry.year === +selectedYear);
        if (selectedMaterial && selectedMaterial !== "All Materials") filteredData = filteredData.filter(entry => entry.material === selectedMaterial);
        if (selectedPlant && selectedPlant !== "All Plants") filteredData = filteredData.filter(entry => entry.plant === selectedPlant);
        
        return Object.values(reduceData(filteredData));
    }

    function renderChart(chartData) {
        const ctx = document.getElementById('inventoryChart').getContext('2d');
        if (chart) chart.destroy();

        chart = new Chart(ctx, {
            type: 'bar',
            data: {
                labels: chartData.map(entry => entry.date),
                datasets: [
                    {
                        label: 'On-Shelf Inventory Quantity',
                        data: chartData.map(entry => entry.onShelfInventoryQuantity),
                        backgroundColor: 'rgba(255, 99, 132, 0.5)'
                    },
                    {
                        label: 'In-Transit Inventory Quantity',
                        data: chartData.map(entry => entry.inTransitQuantity),
                        backgroundColor: 'rgba(75, 192, 192, 0.5)'
                    },
                    {
                        label: 'Gross Inventory Quantity (Total)',
                        data: chartData.map(entry => entry.grossInventoryQuantity),
                        type: 'line',
                        borderColor: 'rgba(255, 206, 86, 1)',
                        borderWidth: 2,
                        pointStyle: 'circle',
                        pointRadius: 5,
                        pointBackgroundColor: 'rgba(255, 206, 86, 1)',
                        fill: false
                    }
                ]
            },
            options: {
                scales: {
                    y: {
                        stacked: true,
                        beginAtZero: true,
                        title: {
                            display: true,
                            text: 'Inventory Quantity'
                        }
                    },
                    x: {
                        stacked: true,
                        title: {
                            display: true,
                            text: 'Date'
                        }
                    }
                }
            }
        });
    }

    function handleYearChange(event) {
        selectedYear = event.target.value;
        const filteredData = filterData();
        renderChart(filteredData);
    }

    function handleMaterialChange(event) {
        selectedMaterial = event.target.value;
        const filteredData = filterData();
        renderChart(filteredData);
    }

    function handlePlantChange(event) {
        selectedPlant = event.target.value;
        const filteredData = filterData();
        renderChart(filteredData);
    }
</script>

<style>
    .selection-container {
        display: flex;
    }

    .selection-container > div {
        margin-right: 20px;
    }
</style>

<div class="selection-container">
    <div>
        <label for="yearSelect">Select Year:</label>
        <select id="yearSelect" bind:value={selectedYear} on:change={handleYearChange}>
            <option value="">All Years</option>
            {#each [...new Set(chartData.map(entry => entry.year))] as year}
            <option value={year}>{year}</option>
            {/each}
        </select>
    </div>

    <div>
        <label for="materialSelect">Select Material Key:</label>
        <select id="materialSelect" bind:value={selectedMaterial} on:change={handleMaterialChange}>
            <option value="">All Materials</option>
            {#each [...new Set(chartData.map(entry => entry.material))] as material}
            <option value={material}>{material}</option>
            {/each}
        </select>
    </div>

    <div>
        <label for="plantSelect">Select Plant Key:</label>
        <select id="plantSelect" bind:value={selectedPlant} on:change={handlePlantChange}>
            <option value="">All Plants</option>
            {#each [...new Set(chartData.map(entry => entry.plant))] as plant}
            <option value={plant}>{plant}</option>
            {/each}
        </select>
    </div>
</div>

<canvas id="inventoryChart"></canvas>
