<script>
  import projects from "$lib/projects.json";
  import Project from "$lib/Project.svelte";
  import reading from "$lib/reading.json";
  import ReadingItem from "$lib/ReadingItem.svelte";
</script>
<svelte:head>
  <title>Home</title>
</svelte:head>
<nav> 
        <a class="current" href=".">Home</a>
        <a href="projects">Projects</a>
        <a href="contact">Contact</a>
        <a href="resume">Resume</a>
        <li><a href="https://github.com/kveni12" target="_blank">Github</a></li>
    </nav>
<div class="main-content">
    <div class="about-me">
        <h1> Krishna Parvataneni</h1>
        <p> Hi, I’m Krishna Parvataneni. I’m an undergraduate at MIT studying Artificial Intelligence and Decision-Making, with interests in machine learning, medical imaging, and healthcare innovation. I enjoy building tools that bridge technical rigor with real-world clinical impact.</p>
        <img src="images/krishna.jpg" alt="a picture of me!">
        <br>
    </div>

    <div class="reading-section">
        <h2> What I'm Reading </h2>
        <div class="readings">
            {#each reading as r}
                <ReadingItem data={r} />
            {/each}
        </div>
    </div>
</div>
<h2> Some of my projects:</h2><br>
<div class="projects highlights">
    {#each projects.slice(0, 3) as p}
        <Project data={p} />
    {/each}
</div>

<style>
/* Flex container for about + reading sections */
.main-content {
    display: flex;
    gap: 1em;
    flex-wrap: wrap;       /* wrap for smaller screens */
    padding-bottom: 2em;   /* extra space below the main-content */
}

/* About Me section styling */
.about-me {
    flex: 2 1 500px;       /* wider section */
}

/* Reading section with purple background */
.reading-section {
    flex: 1 1 300px;       /* narrower section */
    background-color: #c9c0d7; /* light purple */
    padding: 1em;
    border-radius: 8px;
}

/* Make the reading cards wrap horizontally */
.readings {
    display: flex;
    flex-wrap: wrap;
    gap: 0.5em;
}
</style>