<script>
import { base } from '$app/paths';
import { onMount } from 'svelte';
import * as d3 from 'd3';

import BarHorizontal from '$lib/BarHorizontal.svelte';

let locData = [];
let languageData = [];
let commits = [];

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