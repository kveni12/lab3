<script>
import { base } from '$app/paths';
import { onMount } from 'svelte';
import * as d3 from 'd3';

import BarHorizontal from '$lib/BarHorizontal.svelte';

let locData = [];
let languageData = [];

onMount(async () => {

    locData = await d3.csv(`${base}/loc.csv`, row => ({
        ...row,
        line: Number(row.line),
        length: Number(row.length),
        depth: Number(row.depth)
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
});
</script>
<svelte:head>
  <title>Meta</title>
</svelte:head>
<h1>Meta</h1>

{#if languageData.length > 0}

<BarHorizontal data={languageData} />

{/if}