<script>
    // IMPORT LIBS
    import { select } from 'd3-selection';
    // https://www.npmjs.com/package/d3-shape
    // import { line, curveNatural } from 'd3-shape';
    import { line, curveNatural } from "https://cdn.jsdelivr.net/npm/d3-shape@3/+esm";
    import { extent, group, max } from 'd3-array';
    import { scaleLinear, scaleUtc } from 'd3-scale';
    import AxisX from '$components/AxisX.svelte';
    import AxisY from '$components/AxisY.svelte';
    import Tooltip from '$components/Tooltip.svelte';
	import { fade } from 'svelte/transition';
	import { afterUpdate, onMount } from 'svelte';

    // PROPERTIES //
    export let data;
    export let value;

    // VARS // 
    let tooltipData;
    let height = 400;
	let width = 400;
	const margin = {
		top: 20,
		right: 20,
		bottom: 30,
		left: 20 
	};
    const PATH_ID = '2023-2024';
    const UTC_START_YEAR = 2000;
    const UTC_END_YEAR = 2001;

    // FUNCTIONS //
    function intraYear(date) {
        date = new Date(+date);

        const month = date.getUTCMonth();
        if (month > 9) {
            date.setUTCFullYear(UTC_START_YEAR);
        } else {
            date.setUTCFullYear(UTC_END_YEAR);
        }
        
        return date;
    }

    function getPathEnd(pathId) {
        const path = select(`#season-${pathId}`);
        // console.log(path.node())
        if (!path.node()) return;

        // get total length of path
        const totalLength = path.node().getTotalLength();

        // get end coords
        const endPoint = path.node().getPointAtLength(totalLength);
        return endPoint;
    }

    // DYNAMIC VARS //
    // margins
    $: innerHeight = height - margin.top - margin.bottom;
    $: innerWidth = width - margin.left - margin.right;

    // annotation x/y positions
    $: coords = getPathEnd(PATH_ID);
    
    // scales
    $: xScale = scaleUtc()
        .domain([Date.UTC(2000,10,1), Date.UTC(2001,4,1)])
        .range([0, innerWidth])
        .nice();
    $: yScale = scaleLinear()
        .domain([0, max(data, d => d.value)])
        .range([innerHeight, 0])
    
        // line generator
    $: lineGenerator = line()
        .defined(d => !isNaN(d.value))
        .x(d => xScale(intraYear(d.date)))
        // .x(d => xScale(d.date))
        .y(d => yScale(d.value));
    
    // call every time the data changes
    $: {
        coords = getPathEnd(PATH_ID);
        annotatePath(PATH_ID);
    }

    // annotate
    function annotatePath(pathId) {
        // console.log('ANNOTATE!')

        coords = getPathEnd(pathId);
        if (!coords) return;

        select('#lines text').remove(); // Remove any existing text elements
        // redraw them
        select('#lines')
            .append('text')
            .attr('x', coords.x)
            .attr('y', coords.y)
            .attr('dy', -8)
            .attr('text-anchor', 'middle')
            .style('fill', '#0062A3')
            .style('font-family', 'BentonSansCond-Bold')
            .text('This season');
}
    
    // call annotatePath after all path have been updated
    afterUpdate(() => {
        coords = getPathEnd(PATH_ID);
        annotatePath(PATH_ID);
    });
</script>

<div class="chart-container" bind:clientWidth={width}>
    <svg height={height} width={width}>
        <g class="inner-chart" transform="translate({margin.left}, {margin.top})">
            <!-- svelte-ignore a11y-no-static-element-interactions -->
            <g id="lines" on:mouseleave={() => tooltipData = null}>
                {#each group(data, d => d.group) as d, i}
                    <g>
                        <!-- svelte-ignore a11y-no-static-element-interactions -->
                        <!-- svelte-ignore a11y-no-noninteractive-tabindex -->
                        <path d={lineGenerator(d[1])} 
                            fill="none" 
                            id={`season-${d[0]}`}
                            transition:fade
                            stroke={d[0] === '2023-2024' ? "#0062A3" : "#D1D2D4"} 
                            stroke-width={d[0] === '2023-2024' ? "4" : "2"}
                            stroke-linecap="round"
                            tabindex="0"
                            on:mouseover={(e) => {
                                tooltipData = {
                                    year:d[0],
                                    values:d[1],
                                    xPos: e.clientX,
                                    yPos: e.clientY
                                };
                            }}
                            on:focus={(e) => {
                                tooltipData = {
                                    year:d[0],
                                    values:d[1],
                                    xPos: e.clientX,
                                    yPos: e.clientY
                                };
                            }}
                        />
                    </g>
                {/each}

                {console.log('LINE LOOP DONE')}
                
                <!-- HIGHLIGHT LINES ON MOUSEOVER -->
                {#if tooltipData && tooltipData.year !== '2023-2024'}
                    {#key tooltipData.year}
                        <path d={lineGenerator(tooltipData.values)} 
                            fill="none" 
                            stroke="#009775"
                            stroke-width="3"
                            transition:fade
                        />
                    {/key}
                {/if}
            </g>

            <!-- AXES -->
            <AxisX height={height} width={width} {xScale} />
            <AxisY {width} {yScale} />
        </g>
    </svg>
    {#if tooltipData}
        <Tooltip data={tooltipData} {width}/>
    {/if}
</div>

<style>
    :global(.tick text, .axis-title) {
        color: #231F20;
        font-size: 0.9rem;
        user-select: none;
    }
    /* path already has outline when focused */
	path:focus {
		outline: none;
	}
</style>