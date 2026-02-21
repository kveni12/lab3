<script>
  import projects from "$lib/projects.json";
  import Project from "$lib/Project.svelte";
</script>
<nav>
        <a href="../">Home</a>
        <a class="current" href="../projects">Projects</a>
        <a href="../contact">Contact</a>
        <a href="../resume">Resume</a>
        <a href="https://github.com/kveni12" target="_blank">Github</a>
    </nav>
<svelte:head>
  <title>{projects.length} Projects</title>
</svelte:head>
<!-- <pre>{ JSON.stringify(projects, null, "\t") }</pre> -->
    <h1>{projects.length} Projects</h1>
    <div class="projects">
      {#each projects as p}
        <Project data={p}/>
      {/each}
    </div>