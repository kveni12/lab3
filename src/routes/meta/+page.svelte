<script>
import { base } from '$app/paths';
import { onMount } from 'svelte';
import * as d3 from 'd3';
import {
    computePosition,
    autoPlacement,
    offset,
} from '@floating-ui/dom';

import BarHorizontal from '$lib/BarHorizontal.svelte';

let locData = [];
let languageData = [];
let commits = [];

// dimensions
let width = 1000, height = 600;
let margin = { top: 20, right: 30, bottom: 40, left: 50 };

// usable area
let usableArea = {
    top: margin.top,
    right: width - margin.right,
    bottom: height - margin.bottom,
    left: margin.left
};
usableArea.width = usableArea.right - usableArea.left;
usableArea.height = usableArea.bottom - usableArea.top;

// tooltip state
let hoveredIndex = -1;
$: hoveredCommit = commits[hoveredIndex] ?? hoveredCommit ?? {};

let commitTooltip;
let tooltipPosition = { x: 0, y: 0 };

// scales
$: [minDate, maxDate] = d3.extent(commits.map(d => d.datetime));

$: maxDatePlusOne = maxDate ? new Date(maxDate) : new Date();
$: maxDatePlusOne.setDate(maxDatePlusOne.getDate() + 1);

$: xScale = d3.scaleTime()
    .domain([minDate, maxDatePlusOne])
    .range([usableArea.left, usableArea.right])
    .nice();

$: yScale = d3.scaleLinear()
    .domain([24, 0])
    .range([usableArea.bottom, usableArea.top]);

$: [minLines, maxLines] = d3.extent(commits.map(d => d.totalLines));

$: rScale = d3.scaleSqrt()
    .domain([minLines, maxLines])
    .range([5, 30]);

// axes
let xAxis, yAxis, yAxisGridlines;

$: if (xAxis && yAxis && yAxisGridlines) {
    d3.select(xAxis).call(d3.axisBottom(xScale));

    d3.select(yAxis).call(
        d3.axisLeft(yScale)
          .tickFormat(d => String(d % 24).padStart(2, "0") + ":00")
    );

    d3.select(yAxisGridlines).call(
        d3.axisLeft(yScale)
          .tickFormat("")
          .tickSize(-usableArea.width)
    );
}

// interaction
async function dotInteraction(index, evt) {
    let hoveredDot = evt.target;

    if (evt.type === "mouseenter") {
        hoveredIndex = index;

        if (!commitTooltip) return;

        tooltipPosition = await computePosition(hoveredDot, commitTooltip, {
            strategy: "fixed",
            middleware: [
                offset(5),
                autoPlacement()
            ],
        });
    } else if (evt.type === "mouseleave") {
        hoveredIndex = -1;
    }
}

// data loading
onMount(async () => {

    locData = await d3.csv(`${base}/loc.csv`, row => ({
        ...row,
        line: Number(row.line),
        depth: Number(row.depth),
        length: Number(row.length),
        date: new Date(row.date + "T00:00" + row.timezone),
        datetime: new Date(row.datetime)
    }));

    const grouped = d3.rollups(
        locData,
        v => v.length,
        d => d.type
    );

    languageData = grouped.map(([label, value]) => ({
        label,
        value
    })).sort((a,b)=>d3.descending(a.value,b.value));

    commits = d3.groups(locData, d => d.commit).map(([commit, lines]) => {
        let first = lines[0];
        let {author, date, time, timezone, datetime} = first;

        return {
            id: commit,
            url: "https://github.com/kveni12/lab3/commit/" + commit,
            author, date, time, timezone, datetime,
            hourFrac: datetime.getHours() + datetime.getMinutes() / 60,
            totalLines: lines.length,
            lines
        };
    });

    // sort so small dots render on top
    commits = d3.sort(commits, d => -d.totalLines);
});
</script>

<svelte:head>
  <title>Meta</title>
</svelte:head>

<h1>Meta</h1>

{#if languageData.length > 0}
<BarHorizontal data={languageData} />
{/if}

<h3>Commits by time of day</h3>

<svg viewBox="0 0 {width} {height}">
    <!-- x axis -->
    <g transform="translate(0, {usableArea.bottom})" bind:this={xAxis} />

    <!-- gridlines -->
    <g class="gridlines"
       transform="translate({usableArea.left}, 0)"
       bind:this={yAxisGridlines} />

    <!-- y axis -->
    <g transform="translate({usableArea.left}, 0)" bind:this={yAxis} />

    <!-- dots -->
    <g class="dots">
    {#each commits as commit, index}
        <circle
            cx={xScale(commit.datetime)}
            cy={yScale(commit.hourFrac)}
            r={rScale(commit.totalLines)}
            fill="steelblue"
            fill-opacity="0.6"
            style="cursor: pointer"

            on:mouseenter={evt => dotInteraction(index, evt)}
            on:mouseleave={evt => dotInteraction(index, evt)}
        />
    {/each}
    </g>
</svg>

<dl
    class="info tooltip"
    bind:this={commitTooltip}
    hidden={hoveredIndex === -1}
    style="top: {tooltipPosition.y}px; left: {tooltipPosition.x}px"
>
    <dt>Commit</dt>
    <dd>
        <a href={hoveredCommit.url} target="_blank">
            {hoveredCommit.id}
        </a>
    </dd>

    <dt>Date</dt>
    <dd>
        {hoveredCommit.datetime?.toLocaleString("en", { dateStyle: "full" })}
    </dd>

    <dt>Time</dt>
    <dd>
        {hoveredCommit.datetime?.toLocaleString("en", { timeStyle: "short" })}
    </dd>

    <dt>Author</dt>
    <dd>{hoveredCommit.author}</dd>

    <dt>Lines edited</dt>
    <dd>{hoveredCommit.totalLines}</dd>
</dl>

<style>
svg {
    overflow: visible;
}

.gridlines {
    stroke-opacity: .2;
}

circle {
    transition: 200ms;
}

circle:hover {
    fill: darkgreen;
}

/* optional focus effect */
.dots:hover circle {
    opacity: 0.2;
}

.dots circle:hover {
    opacity: 1;
}

dl.info {
    display: grid;
    grid-template-columns: auto 1fr;
    gap: 4px 8px;
    margin: 0;

    transition-duration: 500ms;
    transition-property: opacity, visibility;
}

dl.info dt {
    font-weight: normal;
    color: #666;
}

dl.info dd {
    margin: 0;
    font-weight: bold;
}

dl.info[hidden]:not(:hover, :focus-within) {
    opacity: 0;
    visibility: hidden;
}

.tooltip {
    position: fixed;

    background-color: oklch(100% 0% 0 / 80%);
    backdrop-filter: blur(6px);

    padding: 10px 12px;
    border-radius: 8px;

    box-shadow: 0 4px 20px rgba(0,0,0,0.15);
}
</style>