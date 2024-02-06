<script>
    import { each } from "svelte/internal";
    import { utcFormat } from 'd3-time-format';

    export let height;
    export let width;
    export let xScale;

    const formatDate = utcFormat('%b');

    $: ticks = xScale.ticks(4);
</script>

<g class="axis x">
    {#each ticks as tick, index}
        <g class="tick" transform="translate({xScale(tick)}, 0)">
            <text x="0" y={height} dy="-35"
                dominant-baseline="middle"
                text-anchor={index == 0 ? "start" : "middle"}
                >{formatDate(tick)}
                {formatDate(tick) == "May" ? "" : "."}
            </text>

            <!-- <line x1="0" x2="0" y1="0" y2={height - 40} stroke="hsla(214, 15%, 91%, 1"></line> -->
        </g>
    {/each}
    <!-- <text class="axis-title" x={width} y={height} dy="4"
        dominant-baseline="hanging"
        text-anchor="end"
    >x label copy</text> -->
</g>