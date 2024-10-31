<script>
    import * as d3 from "d3";
    import { legendColor } from "d3-svg-legend";

    export let data;
    export let fullData;

    let tooltip;
    let tooltipData = "";

    let width = 500;
    let height = 500;

    let proj = d3.geoMercator()
        .scale(40000)
        .center([-87.6298, 41.8781])
        .translate([width-200, height-300]);
    let path = d3.geoPath().projection(proj);

    // $: scale = d3.scaleOrdinal(d3.schemeDark2)
        // .domain(d3.extent(data.map((d) => +d.properties.name10)));
    $: daScale = d3.scaleSequential(d3.interpolateGreens)
       .domain(d3.extent(data.map((d) => +d.properties.uninsuredRate)));

    // $: daScale = d3.scaleSequential(d3.interpolatePiYG)
        // .domain([d3.min(data.map((d) => +d.properties.population)), 
        // d3.median(data.map((d) => +d.properties.population)), 
        // d3.max(data.map((d) => +d.properties.population))]);

    let mapLegend;
    $: {
        const colorLegend = legendColor()
            .shape("rect")
            .cells(9)
            .scale(daScale);

        d3.select(mapLegend)
            .call(colorLegend);
    }

    function handleMouseOver(event, d) {
        console.log(d);
        if (tooltip){
            tooltipData = `Percentage Population of Veterans: ${d.properties.vetStatus}\nUninsured Rate: ${d.properties.uninsuredRate}`;
            tooltip.style.opacity = 1;
            tooltip.style.left = `${event.pageX + 5}px`;
            tooltip.style.top = `${event.pageY + 5}px`;
        }
    }

    function handleMouseOut() {
        tooltip.style.opacity = 0;
    }

</script>

<main>
    <div bind:this={tooltip} class="tooltip"></div>
    <svg {width} {height}> 
        {#each data as d}
            <path style = "fill: {daScale(+d.properties.uninsuredRate)};" 
                d={path(d)}
                on:mouseover={(e) => handleMouseOver(e, d)}
                on:focus={(e) => handleMouseOver(e, d)}
                on:mouseout={handleMouseOut}
                on:blur={handleMouseOut}
                />
        {/each}
        {#each fullData as d}
            <path class = "geooverlay"
            d={path(d)}/>
        {/each}

        <g transform="translate({width - 100}, 50)"
              bind:this={mapLegend} />
    </svg>
</main>

<style>
    .geooverlay{
        stroke: grey;
        stroke-width: 1px;
        fill: none;
    }
    .tooltip {
        position: absolute;
        background-color: white;
        border: 1px solid black;
        padding: 5px;
        pointer-events: none;
        opacity: 1;
        transition: opacity 0.2s;
    }
</style>