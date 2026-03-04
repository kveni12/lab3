<script>
  import projects from "$lib/projects.json";
  import Project from "$lib/Project.svelte";
  import ProjectNarrative from "$lib/ProjectNarrative.svelte";

  let years = projects.map(proj => proj.year);
  let range = Math.max(...years) - Math.min(...years) + 1;
</script>

<svelte:head>
  <title>{projects.length} Projects</title>
</svelte:head>

<h1>{projects.length} Projects over {range} Years</h1>

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