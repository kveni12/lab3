<script>
import * as d3 from "d3";

export let data = [];

let width = 500;
let height = 350;

let margin = { top: 50, right: 20, bottom: 50, left: 70 };

let innerWidth = width - margin.left - margin.right;
let innerHeight = height - margin.top - margin.bottom;

$: xScale = d3.scaleBand()
  .domain(data.map(d => d.label))
  .range([0, innerWidth])
  .padding(0.25);

$: maxVal = d3.max(data, d => d.value) || 1;

$: yScale = d3.scaleLinear()
  .domain([0, maxVal + 1])
  .range([innerHeight, 0])
  .nice();

$: colorScale = d3.scaleOrdinal(d3.schemeTableau10)
  .domain(data.map(d => d.label));

let xAxis;
let yAxis;

$: if (xAxis && yAxis) {
  d3.select(xAxis).call(d3.axisBottom(xScale));

  d3.select(yAxis).call(
    d3.axisLeft(yScale)
      .ticks(maxVal)
      .tickFormat(d => Number.isInteger(d) ? d : "")
  );
}
</script>

<div class="container">

<svg viewBox={`0 0 ${width} ${height}`}>

  <!-- TITLE -->
  <text
    x={width/2}
    y={25}
    text-anchor="middle"
    class="chart-title"
  >
    Projects per Year
  </text>

  <!-- CHART AREA -->
  <g transform={`translate(${margin.left},${margin.top})`}>

    <!-- Y AXIS -->
    <g bind:this={yAxis}></g>

    <!-- Y LABEL -->
    <text
      transform={`rotate(-90)`}
      x={-innerHeight/2}
      y={-50}
      text-anchor="middle"
      class="axis-label"
    >
      Number of Projects
    </text>

    <!-- BARS -->
    {#each data as d}
      <rect
        x={xScale(d.label)}
        y={yScale(d.value)}
        width={xScale.bandwidth()}
        height={innerHeight - yScale(d.value)}
        fill={colorScale(d.label)}
      />
    {/each}

    <!-- X AXIS -->
    <g
      transform={`translate(0,${innerHeight})`}
      bind:this={xAxis}
    ></g>

    <!-- X LABEL -->
    <text
      x={innerWidth/2}
      y={innerHeight + 40}
      text-anchor="middle"
      class="axis-label"
    >
      Year
    </text>

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
}

.container {
    display: flex;
    flex-direction: row;
    align-items: center;   /* centers legend vertically relative to chart */
    justify-content: center;
    gap: 2rem;
}

.legend {
    list-style: none;
    padding: 0;
    margin: 0;
    display: flex;
    flex-direction: column;   /* vertical legend */
    gap: 0.6rem;
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