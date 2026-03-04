<script>
  import projects from "$lib/projects.json";
  import Project from "$lib/Project.svelte";
  import reading from "$lib/reading.json";
  import ReadingItem from "$lib/ReadingItem.svelte";
  import {onMount} from "svelte";
  let githubData = null;
  let loading = true;
  let error = null;
  onMount(async() => {
    try{
        console.log("Page has been mounted!");
        let response = await fetch("https://api.github.com/users/kveni12");
        console.log(response);
        githubData = await response.json();
        console.log(githubData);
    } catch (err) { // if the "try" block runs into an error, cancel excecution and run this code instead
        error = err;
    }
    loading = false; // don't forget to add this line!
    })
</script>
<svelte:head>
  <title>Krishna Parvataneni's Portfolio: Home</title>
</svelte:head>
<div class="main-content">
    <div class="about-me">
        <h1> Krishna Parvataneni</h1>
        <p> Hi, I’m Krishna Parvataneni. I’m an undergraduate at MIT studying Artificial Intelligence and Decision-Making, with interests in machine learning, medical imaging, and healthcare innovation. I enjoy building tools that bridge technical rigor with real-world clinical impact.</p>
        <img src="images/krishna.jpg" alt="a picture of me!">
        <br>
    </div>

    <div class="reading-section">
        <h2 class="reading-title"> What I'm Reading </h2>
        <div class="readings">
            {#each reading as r}
                <ReadingItem data={r} />
            {/each}
        </div>
    </div>
</div>
{#if loading}
  <p>Loading...</p>
{:else if error}
  <p>Something went wrong: {error.message}</p>
{:else}
<section class="github">
  <h2>My GitHub Stats</h2>

  <dl class="stats">
    <dt>Followers</dt>
    <dd>{githubData.followers}</dd>

    <dt>Following</dt>
    <dd>{githubData.following}</dd>

    <dt>Public Repositories</dt>
    <dd>{githubData.public_repos}</dd>
  </dl>
</section>
{/if}
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
.reading-title {
    color: black;
}

.stats {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  text-align: center;
}

.stats dt {
  grid-row: 1;
  font-weight: 600;
}

.stats dd {
  grid-row: 2;
  margin: 0;
  font-size: 1.5rem;
}
</style>