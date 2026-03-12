<script>
import * as d3 from "d3";

export let data = [];

let width = 500;
let height = 350;

/* extra right margin for annotation */
let margin = { top: 40, right: 150, bottom: 50, left: 70 };

let innerWidth = width - margin.left - margin.right;
let innerHeight = height - margin.top - margin.bottom;

/* x scale */
$: xScale = d3.scaleBand()
    .domain(data.map(d => d.label))
    .range([0, innerWidth])
    .padding(0.25);

/* max value */
$: maxVal = d3.max(data, d => d.value) || 1;

/* y scale */
$: yScale = d3.scaleLinear()
    .domain([0, maxVal + 1])
    .range([innerHeight, 0])
    .nice();

/* color scale */
$: colorScale = d3.scaleOrdinal(d3.schemeTableau10)
    .domain(data.map(d => d.label));

/* find bar with highest value */
$: maxBar = d3.greatest(data, d => d.value);

let xAxis;
let yAxis;

/* draw axes */
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

    <!-- chart title -->
    <text
        x={width / 2}
        y="25"
        text-anchor="middle"
        class="chart-title">
        Projects per Year
    </text>

    <!-- chart area -->
    <g transform={`translate(${margin.left},${margin.top})`}>

        <!-- y axis -->
        <g bind:this={yAxis}></g>

        <!-- y axis label -->
        <text
            transform="rotate(-90)"
            x={-innerHeight / 2}
            y={-50}
            text-anchor="middle"
            class="axis-label">
            Number of Projects
        </text>

        <!-- bars -->
        {#each data as d}
            <rect
                x={xScale(d.label)}
                y={yScale(d.value)}
                width={xScale.bandwidth()}
                height={innerHeight - yScale(d.value)}
                fill={colorScale(d.label)}
            />
        {/each}

        <!-- annotation -->
        {#if maxBar}

            <!-- highlight rectangle -->
            <rect
                x={xScale(maxBar.label)}
                y={yScale(maxBar.value)}
                width={xScale.bandwidth()}
                height={innerHeight - yScale(maxBar.value)}
                fill="none"
                stroke="currentColor"
                stroke-width="2"
            />

            <!-- leader line -->
            <line
                x1={xScale(maxBar.label) + xScale.bandwidth()}
                y1={yScale(maxBar.value) + (innerHeight - yScale(maxBar.value)) / 2}
                x2={xScale(maxBar.label) + xScale.bandwidth() + 30}
                y2={yScale(maxBar.value) + (innerHeight - yScale(maxBar.value)) / 2}
                stroke="currentColor"
                stroke-width="1"
            />

            <!-- annotation text -->
            <text
                x={xScale(maxBar.label) + xScale.bandwidth() + 35}
                y={yScale(maxBar.value) + (innerHeight - yScale(maxBar.value)) / 2}
                dominant-baseline="middle"
                class="annotation">
                Year with most projects ({maxBar.label})
            </text>

        {/if}

        <!-- x axis -->
        <g
            transform={`translate(0,${innerHeight})`}
            bind:this={xAxis}>
        </g>

        <!-- x axis label -->
        <text
            x={innerWidth / 2}
            y={innerHeight + 40}
            text-anchor="middle"
            class="axis-label">
            Year
        </text>

    </g>

</svg>

<!-- vertical legend -->
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

/* chart + legend layout */
.container {
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: center;
    gap: 2rem;
}

/* legend */
.legend {
    list-style: none;
    padding: 0;
    margin: 0;
    display: flex;
    flex-direction: column;
    gap: 0.6rem;
}

.swatch {
    width: 12px;
    height: 12px;
    background: var(--color);
    display: inline-block;
    margin-right: 6px;
}

li {
    display: flex;
    align-items: center;
    font-size: 13px;
}

/* title */
.chart-title {
    font-size: 18px;
    font-weight: 600;
}

/* axis labels */
.axis-label {
    font-size: 13px;
    fill: #444;
}

/* annotation style */
.annotation {
    font-size: 11px;
    fill: black;
    font-style: italic;
}

</style>