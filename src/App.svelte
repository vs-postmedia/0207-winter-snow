<script>
    // COMPONENTS 
    import { csv } from "d3-fetch";
    import { onMount } from 'svelte';
    import { group } from 'd3-array';
    import '@carbon/charts-svelte/styles.css';
    // import { LineChart } from '@carbon/charts-svelte';
    import Chart from "$components/Chart.svelte";
    import Select from "svelte-select"; // https://github.com/rob-balfre/svelte-select

    // DATA
    import { menuItems } from "$data/menu-items";
    const dataUrl = 'https://vs-postmedia-data.sfo2.digitaloceanspaces.com/misc/ns-mountains-snowfall.csv';


    // VARIABLES
    let chartData, data, value;
    const defaultSelectValue = menuItems[0].value;

    // REACTIVE VARIABLES
    $: value, updateData(value);

    // FUNCTIONS
    function updateData(value) {
        // console.log(value)
        if (!value || !value.value) return;
        chartData = data.filter(d => d.station.toLowerCase() === value.value.toLowerCase());
    }

    function getDefaultDisplay() {
        const urlParams = new URLSearchParams(window.location.search);
        const station = urlParams.get('station');
        return station;
    }

    async function init() {
        console.log("INIT!");

        // fetch data
        const csvData = await csv(dataUrl);

        // process data
        data = csvData.map(d => {

            return {
                date: new Date(d.local_date),
                value: parseFloat(d.cumulative_snow),
                group: d.season,
                station: d.station
            }  
        }); 

        // if there's a 'station' url param, use that
        const defaultDisplay = getDefaultDisplay();
        // otherwise use the default display selector value
		value = defaultDisplay ? defaultDisplay : defaultSelectValue;
    }

    $: chartData; 

    onMount(init);
</script>

<header>
    <h1>Where’s winter? See how this year’s snow stacks up on B.C. slopes</h1>
    <p class="subhead">Cumulative daily snowfall at select B.C. ski resorts each season since 2000.</p>
</header>

<main>
    <Select items={menuItems}
        bind:value
        change={updateData}
        placeholder="Pick a city..."
		showChevron="true"
		listOpen={false}
    />
    
    {#if chartData}
        <!-- <LineChart {data} {options} /> -->
        <Chart 
            data={chartData}
            value={value}
        />
    {/if}
</main>

<footer>
    <p class="note">NOTE: Does not account for snow making or snow loss from warm weather or rain. Snowfall levels are based on data from nearby weather stations and may not align with measurements from resort operators.</p>
    <p class="source">Source:  <a href="https:vancouversun.com" target="_blank">Environment and Climate Change Canada</a></p>
</footer>
  
<style>
    @import '$css/normalize.css';
    @import '$css/fonts.css';
    @import '$css/colors.css';
    @import '$css/app.css';

    #header {
		margin-bottom: 2rem;
	}
	#header > h1 {
		text-align: center;
	}
	#header .subhead {
		margin: 0 auto;
		max-width: 525px;
		text-align: center;
	}

    footer .source {
        font-family: 'BentonSansCond-Regular', sans;
    }

    /* COMBOBOX SELECTOR */
  	:global(.svelte-select) {
		margin: 1rem auto !important;
		max-width: 250px;
  	}
  	:global(input:focus) {
		outline: none;
  	}

	:global(
		.svelte-select .selected-item,
		.svelte-select .item,
		.svelte-select input
	) {
		font-family: 'BentonSansCond-Regular', sans;
	}
</style>
