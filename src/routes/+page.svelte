<script>
  import projects from "$lib/projects.json";
  import Project from "$lib/Project.svelte";
  import reading from "$lib/reading.json";
  import ReadingItem from "$lib/ReadingItem.svelte";
  import {onMount} from "svelte";
  import { base } from '$app/paths';
  import * as d3 from 'd3';
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

    <dt>Total <abbr title="Lines of code">LOC</abbr></dt>
    <dd>{locData.length}</dd>
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