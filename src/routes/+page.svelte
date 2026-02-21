<script>
  import projects from "$lib/projects.json";
  import Project from "$lib/Project.svelte";
</script>
<nav> 
        <a class="current" href=".">Home</a>
        <a href="projects">Projects</a>
        <a href="contact">Contact</a>
        <a href="resume">Resume</a>
        <li><a href="https://github.com/kveni12" target="_blank">Github</a></li>
    </nav>
    <h1> Krishna Parvataneni</h1>
    <p> Hi, I’m Krishna Parvataneni. I’m an undergraduate at MIT studying Artificial Intelligence and Decision-Making, with interests in machine learning, medical imaging, and healthcare innovation. I enjoy building tools that bridge technical rigor with real-world clinical impact.</p>
    <img src="images/krishna.jpg" alt="a picture of me!">
    <h2> Some of my projects:</h2>
    <div class="projects highlights">
    {#each projects.slice(0, 3) as p}
        <Project data={p} />
    {/each}
    </div>