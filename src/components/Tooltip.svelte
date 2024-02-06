<script>
    // PROPS
    export let data;
    export let width;

    // LIBS
    import { fly, fade } from 'svelte/transition';

    // VARS
    let tooltipWidth;
    const xNudge = 5;
    const yNudge = 5;

    // DYNAMIC VARA
    let xPosition = 125;
    let yPosition = 60;

    // $: console.log(data)
    // style="left: {xPosition}px; position: absolute; top: {yPosition}px;"
</script>

<div class="tooltip"
    bind:clientWidth={tooltipWidth}
    out:fade
    style="position:absolute; text-align:center; top:0; width:100%;"
    >
    <h1>{data.year}</h1>
</div>


<style>
    #app .tooltip h1 {
        color: #009775;
    }
</style>