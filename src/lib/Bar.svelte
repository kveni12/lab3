<script>
import * as d3 from 'd3';

let width = 400;
let height = 300;
let data = [
    { label: "A", value: 10 },
    { label: "B", value: 20 }
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
</script>
<svg viewBox="0 0 {width} {height}">
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

<style>
    svg {
    max-width: 100%;
    height: auto;
    overflow: visible;
}
</style>