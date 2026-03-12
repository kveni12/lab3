<script>
import * as d3 from "d3";

export let data = [];

let width = 600;
let height = 350;

let margin = { top: 40, right: 150, bottom: 50, left: 120 };

let innerWidth = width - margin.left - margin.right;
let innerHeight = height - margin.top - margin.bottom;

/* x scale (values) */
$: maxVal = d3.max(data, d => d.value) || 1;

$: xScale = d3.scaleLinear()
    .domain([0, maxVal])
    .range([0, innerWidth])
    .nice();

/* y scale (categories) */
$: yScale = d3.scaleBand()
    .domain(data.map(d => d.label))
    .range([0, innerHeight])
    .padding(0.25);

/* color scale */
$: colorScale = d3.scaleOrdinal(d3.schemePastel1)
    .domain(data.map(d => d.label));

/* find largest bar */
$: maxBar = d3.greatest(data, d => d.value);

let xAxis;
let yAxis;

/* draw axes */
$: if (xAxis && yAxis) {

    d3.select(xAxis).call(d3.axisBottom(xScale));

    d3.select(yAxis).call(d3.axisLeft(yScale));
}
</script>

<div class="container">

<svg viewBox={`0 0 ${width} ${height}`}>

<!-- title -->
<text
    x={width/2}
    y="25"
    text-anchor="middle"
    class="chart-title">
    Lines of Code by Language
</text>

<g transform={`translate(${margin.left},${margin.top})`}>

<!-- y axis -->
<g bind:this={yAxis}></g>

<!-- bars -->
{#each data as d}
    <rect
        x="0"
        y={yScale(d.label)}
        width={xScale(d.value)}
        height={yScale.bandwidth()}
        fill={colorScale(d.label)}
    />
{/each}

<!-- annotation -->
{#if maxBar}

<rect
    x="0"
    y={yScale(maxBar.label)}
    width={xScale(maxBar.value)}
    height={yScale.bandwidth()}
    fill="none"
    stroke="black"
    stroke-width="2"
/>

<line
    x1={xScale(maxBar.value)}
    y1={yScale(maxBar.label) + yScale.bandwidth()/2}
    x2={xScale(maxBar.value) + 30}
    y2={yScale(maxBar.label) + yScale.bandwidth()/2}
    stroke="black"
/>

<text
    x={xScale(maxBar.value) + 35}
    y={yScale(maxBar.label) + yScale.bandwidth()/2}
    dominant-baseline="middle"
    class="annotation">
    Most lines of code
</text>

{/if}

<!-- x axis -->
<g
    transform={`translate(0,${innerHeight})`}
    bind:this={xAxis}
/>

<!-- x label -->
<text
    x={innerWidth/2}
    y={innerHeight + 40}
    text-anchor="middle"
    class="axis-label">
    Lines of Code
</text>

</g>

</svg>

<!-- legend -->
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

.container{
display:flex;
flex-direction:row;
align-items:center;
gap:2rem;
}

svg{
max-width:100%;
height:auto;
}

.chart-title{
font-size:18px;
font-weight:600;
}

.axis-label{
font-size:13px;
fill:#444;
}

.legend{
list-style:none;
padding:0;
margin:0;
display:flex;
flex-direction:column;
gap:0.5rem;
}

.swatch{
width:12px;
height:12px;
background:var(--color);
display:inline-block;
margin-right:6px;
}

li{
display:flex;
align-items:center;
font-size:13px;
}

.annotation{
font-size:11px;
font-style:italic;
}

</style>