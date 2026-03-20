<script>
import * as d3 from "d3";

export let data = [];
export let title = "";

let width = 500;
let height = 250;

let margin = { top: 30, right: 80, bottom: 40, left: 100 };

let innerWidth = width - margin.left - margin.right;
let innerHeight = height - margin.top - margin.bottom;

$: maxVal = d3.max(data, d => d.value) || 1;

$: xScale = d3.scaleLinear()
    .domain([0, maxVal])
    .range([0, innerWidth])
    .nice();

$: yScale = d3.scaleBand()
    .domain(data.map(d => d.label))
    .range([0, innerHeight])
    .padding(0.25);

$: colorScale = d3.scaleOrdinal(d3.schemePastel1)
    .domain(data.map(d => d.label));

$: maxBar = d3.greatest(data, d => d.value);

let xAxis, yAxis;

$: if (xAxis && yAxis) {
    d3.select(xAxis).call(
        d3.axisBottom(xScale)
            .ticks(Math.min(maxVal, 10))
    );

    d3.select(yAxis).call(d3.axisLeft(yScale));
}
</script>

<div class="container">

<svg viewBox={`0 0 ${width} ${height}`}>

<text x={width/2} y="20" text-anchor="middle" class="chart-title">
    {title}
</text>

<g transform={`translate(${margin.left},${margin.top})`}>

<g bind:this={yAxis}></g>

{#each data as d}
<rect
    x="0"
    y={yScale(d.label)}
    width={xScale(d.value)}
    height={yScale.bandwidth()}
    fill={colorScale(d.label)}
/>
{/each}

{#if maxBar}
<text
    x={xScale(maxBar.value) + 10}
    y={yScale(maxBar.label) + yScale.bandwidth()/2}
    dominant-baseline="middle"
    text-anchor="start"
    class="annotation">
    Most lines of code
</text>
{/if}

<g transform={`translate(0,${innerHeight})`} bind:this={xAxis} />

</g>

</svg>

<ul class="legend">
{#each data as d}
<li style="--color:{colorScale(d.label)}">
<span class="swatch"></span>
{d.label} <em>({d.value})</em>
</li>
{/each}
</ul>

</div>

<style>
.container {
    display:flex;
    gap:1.5rem;
    align-items:center;
}

.chart-title { font-size:14px; }
.annotation { font-size:10px; }

.legend {
    list-style:none;
    padding:0;
    font-size:11px;
}

.swatch {
    width:10px;
    height:10px;
    background:var(--color);
    display:inline-block;
    margin-right:5px;
}
</style>