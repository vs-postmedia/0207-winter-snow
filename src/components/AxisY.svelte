<script>
    export let yScale;
    export let width;

    $: ticks = yScale.ticks(4);
</script>

<g class="axis y">
    {#each ticks as tick}
        <g class="tick" transform="translate(0, {yScale(tick)})">
            {#if tick > 0}
                <text y="0" dy="-5" x="0">{tick} cm</text>
            {/if}

            <line x1="0" x2={width} y1="0" y2="0" stroke="hsla(214, 15%, 91%, 1"></line>
        </g>
    {/each}
</g>