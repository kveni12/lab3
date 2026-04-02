<script>
import { onMount } from 'svelte';
import * as d3 from 'd3';
import { computePosition, autoPlacement, offset } from '@floating-ui/dom';

import BarHorizontal from '$lib/BarHorizontal.svelte';
import LineChart from '$lib/LineChart.svelte';

// data
let locData = [];
let commits = [];
let linesByDate = [];

// dimensions
let width = 1000, height = 600;
let margin = { top: 20, right: 30, bottom: 40, left: 50 };

let usableArea = {
    top: margin.top,
    right: width - margin.right,
    bottom: height - margin.bottom,
    left: margin.left
};
usableArea.width = usableArea.right - usableArea.left;
usableArea.height = usableArea.bottom - usableArea.top;

// interaction
let hoveredIndex = -1;
let clickedCommits = [];
let svg;

$: hoveredCommit = commits[hoveredIndex] ?? {};

// tooltip
let commitTooltip;
let tooltipPosition = { x: 0, y: 0 };

// =========================
// BRUSH STATE
// =========================
let brushSelection = null;

function brushed(evt) {
    brushSelection = evt.selection;
}

// =========================
// SCALES
// =========================
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

// =========================
// SELECTION LOGIC (FIXED)
// =========================
function isCommitBrushed(commit) {
    if (!brushSelection) return false;

    const [[x0, y0], [x1, y1]] = brushSelection;

    const x = xScale(commit.datetime);
    const y = yScale(commit.hourFrac);

    return x0 <= x && x <= x1 && y0 <= y && y <= y1;
}

$: brushedCommits = brushSelection
    ? commits.filter(isCommitBrushed)
    : [];

// ✅ KEY FIX: only treat as selection if non-empty
$: selectedCommits =
    (clickedCommits.length || brushedCommits.length)
        ? Array.from(new Set([...clickedCommits, ...brushedCommits]))
        : [];

// =========================
// BAR DATA (FIXED)
// =========================
$: barData = (() => {
    const source =
        selectedCommits.length > 0
            ? selectedCommits.flatMap(d => d.lines)
            : locData;

    const counts = d3.rollup(source, v => v.length, d => d.type);

    return Array.from(counts, ([label, value]) => ({ label, value }));
})();

// =========================
// LINE DATA
// =========================
$: if (locData.length) {
    const rolled = d3.rollups(
        locData,
        v => v.length,
        d => d3.timeDay.floor(d.datetime)
    ).map(([date, count]) => ({ date, count }));

    const [minDate, maxDate] = d3.extent(rolled, d => d.date);

    const allDays = d3.timeDays(
        minDate,
        d3.timeDay.offset(maxDate, 1)
    );

    linesByDate = allDays.map(date => ({
        date,
        count: rolled.find(d => +d.date === +date)?.count ?? 0
    }));
}

// =========================
// BRUSH SETUP
// =========================
$: if (svg) {
    const brush = d3.brush()
        .extent([
            [usableArea.left, usableArea.top],
            [usableArea.right, usableArea.bottom]
        ])
        .on("start brush end", brushed);

    d3.select(svg).call(brush);
    d3.select(svg).selectAll(".dots, .overlay ~ *").raise();
}

// =========================
// INTERACTION
// =========================
async function dotInteraction(index, evt) {
    const commit = commits[index];

    if (evt.type === "mouseenter") {
        hoveredIndex = index;

        tooltipPosition = await computePosition(evt.target, commitTooltip, {
            strategy: "fixed",
            middleware: [offset(5), autoPlacement()],
        });
    }

    if (evt.type === "mouseleave") {
        hoveredIndex = -1;
    }

    if (evt.type === "click") {
        clickedCommits = clickedCommits.includes(commit)
            ? clickedCommits.filter(c => c !== commit)
            : [...clickedCommits, commit];
    }
}

// =========================
// LOAD DATA
// =========================
onMount(async () => {
    locData = await d3.csv(`loc.csv`, d => ({
        ...d,
        datetime: new Date(d.datetime)
    }));

    commits = d3.groups(locData, d => d.commit).map(([commit, lines]) => {
        const first = lines[0];

        return {
            id: commit,
            url: `https://github.com/kveni12/lab3/commit/${commit}`,
            author: first.author,
            datetime: first.datetime,
            hourFrac: first.datetime.getHours() + first.datetime.getMinutes()/60,
            totalLines: lines.length,
            lines
        };
    });

    commits = d3.sort(commits, d => -d.totalLines);
});
</script>

<h1>Meta</h1>

<BarHorizontal
    data={barData}
    title={
        selectedCommits.length > 0
            ? `${selectedCommits.length} Selected Commits`
            : "All Commits Breakdown"
    }
/>

<h3>Commits by time of day</h3>

<svg bind:this={svg} viewBox="0 0 {width} {height}">
    <g class="dots">
    {#each commits as commit, index}
        <circle
            cx={xScale(commit.datetime)}
            cy={yScale(commit.hourFrac)}
            r={rScale(commit.totalLines)}
            fill={selectedCommits.includes(commit)
                ? "var(--color-accent)"
                : "steelblue"}
            fill-opacity="0.6"

            on:mouseenter={e => dotInteraction(index, e)}
            on:mouseleave={e => dotInteraction(index, e)}
            on:click={e => dotInteraction(index, e)}
        />
    {/each}
    </g>
</svg>

<h3 style="text-align:center;">Lines Edited by Day</h3>
<LineChart data={linesByDate} />

<dl bind:this={commitTooltip} hidden={hoveredIndex === -1}
    style="top:{tooltipPosition.y}px; left:{tooltipPosition.x}px">
    <dd>{hoveredCommit.id}</dd>
</dl>

<style>
svg { overflow: visible; }

svg :global(.selection) {
    fill-opacity: 0.1;
    stroke: black;
    stroke-dasharray: 5 3;
}
</style>