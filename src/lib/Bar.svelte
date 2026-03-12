<script>
import * as d3 from 'd3';

let width = 400;
let height = 300;
let data = [
    { label: "A", value: 10 },
    { label: "B", value: 20 },
    { label: "C", value: 15 },
    { label: "D", value: 8 },
    { label: "E", value: 25 }
];
let margin = { top: 20, right: 20, bottom: 30, left: 60 };
let innerWidth  = width  - margin.left - margin.right;
let innerHeight = height - margin.top  - margin.bottom;
$: xScale = d3.scaleBand()
    .domain(data.map(d => d.label))
    .range([0, innerWidth])
    .padding(0.2);

$: yScale = d3.scaleLinear()
    .domain([0, d3.max(data, d => d.value) || 1])
    .range([innerHeight, 0]);

$: colorScale = d3.scaleOrdinal(d3.schemeTableau10)
    .domain(data.map(d => d.label));

let xAxis, yAxis;
$: if (xAxis && yAxis) {
    d3.select(xAxis).call(d3.axisBottom(xScale));
    d3.select(yAxis).call(d3.axisLeft(yScale));
}

</script>
<div class="container">
    <svg viewBox="0 0 {width} {height}">
        <g transform="translate({margin.left}, {margin.top + innerHeight})"
        bind:this={xAxis} />
        <g transform="translate({margin.left}, {margin.top})"
        bind:this={yAxis} />
        <g transform="translate({margin.left}, {margin.top})">
            {#each data as d}
                <rect
                    x={xScale(d.label)}
                    y={yScale(d.value)}
                    width={xScale.bandwidth()}
                    height={innerHeight - yScale(d.value)}
                    fill={colorScale(d.label)}
                />
            {/each}
        </g>
    </svg>
    <ul class="legend">
        {#each data as d}
            <li style="--color: {colorScale(d.label)}">
                <span class="swatch"></span>
                {d.label} <em>({d.value})</em>
            </li>
        {/each}
    </ul>
</div>

<style>
svg {
    max-width: 100%;
    height: auto;
    overflow: visible;
}
.container {
    display: flex;
    flex-direction: column;
    align-items: center;
}
.legend {
    flex: 1;
    list-style: none;
    padding: 0;
    margin: 1em 0 0;
    display: flex;
    gap: 1em;
}
.swatch {
    display: inline-block;
    width: 1em;
    height: 1em;
    background-color: var(--color);
    margin-right: 0.5em;
}
li {
    display: flex;
    align-items: center;
}
</style>