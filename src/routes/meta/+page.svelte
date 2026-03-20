<script>
import { base } from '$app/paths';
import { onMount } from 'svelte';
import * as d3 from 'd3';

import BarHorizontal from '$lib/BarHorizontal.svelte';

let locData = [];
let languageData = [];
let commits = [];
let width = 1000, height = 600;

// scales
$: [minDate, maxDate] = d3.extent(commits.map(d => d.date));
$: maxDatePlusOne = new Date(maxDate);
$: maxDatePlusOne.setDate(maxDatePlusOne.getDate() + 1);
let margin = { top: 20, right: 30, bottom: 40, left: 50 };

let usableArea = {
    top: margin.top,
    right: width - margin.right,
    bottom: height - margin.bottom,
    left: margin.left
};
usableArea.width = usableArea.right - usableArea.left;
usableArea.height = usableArea.bottom - usableArea.top;

$: xScale = d3.scaleTime()
              .domain([minDate, maxDatePlusOne])
              .range([usableArea.left, usableArea.right])
              .nice();

$: yScale = d3.scaleLinear()
              .domain([24, 0])
              .range([usableArea.bottom, usableArea.top]);

$: [minLines, maxLines] = d3.extent(commits.map(d => d.totalLines));

$: rScale = d3.scaleLinear()
    .domain([minLines, maxLines])
    .range([5, 30]);
let xAxis, yAxis;
let yAxisGridlines;
$: {
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

onMount(async () => {

    locData = await d3.csv(`${base}/loc.csv`, row => ({
        ...row,
        line: Number(row.line),
        depth: Number(row.depth),
        length: Number(row.length),
        date: new Date(row.date + "T00:00" + row.timezone),
        datetime: new Date(row.datetime)
    }));
    /* group by language type */
    const grouped = d3.rollups(
        locData,
        v => v.length,
        d => d.type
    );

    languageData = grouped.map(([label, value]) => ({
        label,
        value
    }))
    .sort((a,b)=>d3.descending(a.value,b.value));
    commits = d3.groups(locData, d => d.commit).map(([commit, lines]) => {
        let first = lines[0];
        let {author, date, time, timezone, datetime} = first;
        let ret = {
            id: commit,
            url: "https://github.com/kveni12/lab3/commit/" + commit,
            author, date, time, timezone, datetime,
            hourFrac: datetime.getHours() + datetime.getMinutes() / 60,
            totalLines: lines.length,
            lines: lines
        };

        return ret;
    });
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

    <!-- gridlines (BEFORE y-axis) -->
    <g class="gridlines"
       transform="translate({usableArea.left}, 0)"
       bind:this={yAxisGridlines} />

    <!-- y axis -->
    <g transform="translate({usableArea.left}, 0)" bind:this={yAxis} />

    <!-- dots -->
    <g class="dots">
    {#each commits as commit, index }
        <circle
            cx={ xScale(commit.datetime) }
            cy={ yScale(commit.hourFrac) }
            r={rScale(commit.totalLines)}
            fill="steelblue"
            fill-opacity="0.6"
        />
    {/each}
    </g>
</svg>

<style>
    svg {
        overflow: visible;
    }

    .gridlines {
        stroke-opacity: .2;
    }
</style>