<!-- Scatter.svelte -->
 <script>
    import * as d3 from "d3";

    export let data;
    export let fullData;
    export let var1;
    export let var2;
    export let filter;
    export let update;

    let margin = {top: 10, right: 30, bottom: 30, left: 40};
    let width = 360;
    let height = 200;
    let chartW = width - margin.left - margin.right;
    let chartH = height - margin.top - margin.bottom;

    let brushLayer;
    let xAxis;
    let yAxis;

    const brush = d3.brush()
        .extent([[0, 0], [chartW, chartH]])
        .on("brush", brushed)
        .on("end", brushended);

    function brushed(event) {
        if (event.selection) {
            const [[x0, y0], [x1, y1]] = event.selection;
            filter = [
                [xScale.invert(x0), yScale.invert(y1)],
                [xScale.invert(x1), yScale.invert(y0)]
            ];
            update();
        }
    }

    function brushended(event) {
        if (!event.selection) {
            filter = [];
            update();
        }
    }

    $: xScale = d3.scaleLinear()
        .range([0, chartW])
        .domain(d3.extent(fullData, d => d.properties.vetStatus));
    $: yScale = d3.scaleLinear()
        .range([chartH, 0])
        .domain(d3.extent(fullData, d => d.properties.uninsuredRate));

    $: {
        d3.select(brushLayer)
            .call(brush);
        d3.select(xAxis)
            .call(d3.axisBottom(xScale));
        d3.select(yAxis)
            .call(d3.axisLeft(yScale));
    }
 </script>

 <main>
    <svg {width} {height}>
        <g transform="translate({margin.left}, {margin.top})">
            {#each data as d}
                <circle cx={xScale(d.properties.vetStatus)} 
                cy={yScale(d.properties.uninsuredRate)} 
                r="3"
                fill="steelblue"/>
            {/each}
        </g>
        <!-- Brush Layer -->
        <g transform="translate({margin.left}, {margin.top})" bind:this={brushLayer} />
        <!-- X-Axis -->
        <g transform="translate({margin.left}, {height - margin.bottom})" bind:this={xAxis} />
        <!-- Y-Axis -->
        <g transform="translate({margin.left}, {margin.top})" bind:this={yAxis} />
        <!-- X-Axis Label -->
        <text transform="translate({width / 2}, {height})" text-anchor="middle">
            {var1}
        </text>
        <!-- Y-Axis Label -->
        <text transform="translate({width}, {height / 2}) rotate(-90)" text-anchor="middle">
            {var2}
        </text>
    </svg>
 </main>

 <style>
    circle {
        opacity: 0.7;
    }
</style>
