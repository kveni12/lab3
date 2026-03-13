<script>
  import projects from "$lib/projects.json";
  import Project from "$lib/Project.svelte";
  import ProjectNarrative from "$lib/ProjectNarrative.svelte";
  import Bar from "$lib/Bar.svelte";

  let years = projects.map(proj => proj.year);
  let range = Math.max(...years) - Math.min(...years) + 1;
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  let rawData = [];
  let wrangled = [];
  

  onMount(async () => {
      rawData = await d3.json('lab6_example.json');
      wrangled = d3.rollups(
          rawData,
          v => d3.sum(v, d => d.lines),
          d => d.language
      );
  });
  $: barData = d3.rollups(projects, v => v.length, d => d.year)
    .map(([year, count]) => ({ label: String(year), value: count }));
</script>

<svelte:head>
  <title>{projects.length} Projects</title>
</svelte:head>

<h1>{projects.length} Projects over {range} Years</h1>
<section>
    <h2>Data wrangling result</h2>
    <pre>{JSON.stringify(wrangled, null, 2)}</pre>
</section>
<section>
<Bar data={barData} />
</section>  
<p>
Scroll down to see a timeline of my projects and how they've contributed to my professional and personal life
</p>

<ProjectNarrative projects={projects} />

<p class="outro">
Thanks for scrolling through my project story! Feel free to explore all of the projects at your leisure below.
</p>

<div class="projects">
  {#each projects as p}
    <Project data={p}/>
  {/each}
</div>

<style>
.projects {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px,1fr));
  gap: 1rem;
}

.outro {
  margin-top: 2em;
  font-style: italic;
  color: gray;
  margin-bottom: 2em;
}
</style>