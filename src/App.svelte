<script>
	import * as d3 from "d3";
	import { onMount } from "svelte";
	import Map from "./Map.svelte";
	import Histogram from './Histogram.svelte';
	import Scatter from "./Scatter.svelte";

	let data = [];
    let fullData = [];
	let filter1 = [];
	let filter2 = [];
	let scatterFilter = [];

	let var1 = "vetStatus";
	let var2 = "uninsuredRate";
	// let var3 = "vaCoverage";

	onMount(async function () {
        let table = d3.csv("chi_health_data.csv", (d) => ({
            ...d,
            'Name': d['Name'].substring(6),
            'VET_2018-2022': +d['VET_2018-2022'],
            'VHA_2018-2022': +d['VHA_2018-2022'],
            'UNS_2018-2022': +d['UNS_2018-2022'],
        }));
    
        let geocoord = d3.json('census-tracts.geojson')
            .then((d) => d.features);
        
        await Promise.all([table, geocoord]).then((values) => {
            // console.log(values);
            let table = values[0];
            let geocoord = values[1];
            // console.log(table);
            for (let i = 0; i < geocoord.length; i++) {
                let tract = geocoord[i].properties.name10;
                let found = false;
                let j = 0;
                while (!found && table.length > j) {
                    if (table[j].Name == tract) {
                        found = true;
                        data.push(geocoord[i]);
                        data[data.length - 1].properties["population"] = table[j]['Population'];
                        data[data.length - 1].properties["vetStatus"] = table[j]['VET_2018-2022'];
                        data[data.length - 1].properties["vaCoverage"] = table[j]['VHA_2018-2022'];
                        data[data.length - 1].properties["uninsuredRate"] = table[j]['UNS_2018-2022'];
                    } else {
                        j++;
                    }
                }
            }
            console.log(data);
            fullData = [...data];
        });
    });

	function updateData() {
		if (filter1.length > 0 && filter2.length >0) {
			data = fullData.filter((d) => (d.properties[var1] >= filter1[0] && d.properties[var1] < filter1[1] && d.properties[var2] >= filter2[0] && d.properties[var2] < filter2[1]));
		} else if (filter1.length > 0 && filter2.length == 0) {
			data = fullData.filter((d) => (d.properties[var1] >= filter1[0] && d.properties[var1] < filter1[1]));
		} else if (filter1.length == 0 && filter2.length > 0) {
			data = fullData.filter((d) => (d.properties[var2] >= filter2[0] && d.properties[var2] < filter2[1]));
		} else if (scatterFilter.length == 2) {
			data = fullData.filter((d) => (
				d.properties[var1] >= scatterFilter[0][0] && 
				d.properties[var1] < scatterFilter[1][0] &&
				d.properties[var2] >= scatterFilter[0][1] && 
				d.properties[var2] < scatterFilter[1][1]
			));
		} else {
			data = [...fullData];
		}
	}

</script>

<main>
	<h1>Chicago Veteran Healthcare Census Data</h1>
	<div class="flex-container row">
        <div class="map">
			<Map data={data} fullData={fullData}></Map>
		</div>
		<div class = "flex-container col">
			<!-- Histogram Component -->
			<div class="hist">
				<Histogram data={data} fullData={fullData} variable={var1} bind:filter={filter1} update={updateData}></Histogram>
			</div>
			<!-- Histogram Component -->
			<div class="hist">
				<Histogram data={data} fullData={fullData} variable={var2} bind:filter={filter2} update={updateData}></Histogram>
			</div>
			<!-- Scatter Component -->
			<div class="scatter">
				<Scatter data={data} fullData={fullData} var1={var1} var2={var2} bind:filter={scatterFilter} update={updateData}></Scatter>
			</div>
		</div>
    </div>
</main>

<style>
    .flex-container {
        display: flex;
        justify-content: center;
        height: 100%;
        padding: 15px;
        gap: 5px;
    }

    .flex-container > div {
        padding: 8px;
    }

	.flex-container.row {
		flex-direction: row;
	}

	.flex-container.col {
		flex-direction: column;
	}

    .map {
		width: 500px;
    	height: auto;
    	overflow: visible;
        flex-grow: 1;
    }

	.hist, .scatter {
		flex-grow: 0;
	}

</style>