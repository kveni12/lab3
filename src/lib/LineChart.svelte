<script>
import * as d3 from 'd3';

export let data = [];

// dimensions
let width = 1000, height = 300;
let margin = { top: 20, right: 30, bottom: 40, left: 50 };

let usableArea = {
    top: margin.top,
    right: width - margin.right,
    bottom: height - margin.bottom,
    left: margin.left
};
usableArea.width = usableArea.right - usableArea.left;
usableArea.height = usableArea.bottom - usableArea.top;

// scales
$: xScale = d3.scaleTime()
    .domain(d3.extent(data, d => d.date))
    .range([usableArea.left, usableArea.right]);

$: yScale = d3.scaleLinear()
    .domain([0, d3.max(data, d => d.count)])
    .range([usableArea.bottom, usableArea.top])
    .nice();

// line generator
$: line = d3.line()
    .x(d => xScale(d.date))
    .y(d => yScale(d.count))
    .curve(d3.curveBumpX); // smooth curve

// axes
let xAxis, yAxis;

$: if (xAxis && yAxis) {
    d3.select(xAxis).call(d3.axisBottom(xScale));
    d3.select(yAxis).call(d3.axisLeft(yScale));
}
</script>

<svg viewBox="0 0 {width} {height}">
    <!-- axes -->
    <g transform="translate(0, {usableArea.bottom})" bind:this={xAxis} />
    <g transform="translate({usableArea.left}, 0)" bind:this={yAxis} />

    <!-- line -->
    <path
        d={line(data)}
        fill="none"
        stroke="steelblue"
        stroke-width="2"
    />

    <!-- dots -->
    {#each data as d}
        <circle
            cx={xScale(d.date)}
            cy={yScale(d.count)}
            r="3"
            fill="steelblue"
        />
    {/each}

    <!-- x label -->
    <text
        x={usableArea.left + usableArea.width / 2}
        y={height - 5}
        text-anchor="middle"
        class="axis-label">
        Date
    </text>

    <!-- y label -->
    <text
        x={-(usableArea.top + usableArea.height / 2)}
        y={15}
        text-anchor="middle"
        transform="rotate(-90)"
        class="axis-label">
        Number of Lines Edited
    </text>
</svg>

<style>
svg {
    overflow: visible;
}

.axis-label {
    font-size: 0.8em;
    fill: currentColor;
}
</style>