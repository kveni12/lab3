<script>
import Scrolly from "svelte-scrolly";

export let projects = [];

let sorted_projects = [...projects].sort((a,b)=>a.year-b.year);

let scrollyProgress = 0;

$: activeProjectIdx = Math.min(
  sorted_projects.length - 1,
  Math.floor((scrollyProgress / 100) * sorted_projects.length)
);
</script>

<div class="scrolly-wrapper">

<Scrolly bind:progress={scrollyProgress}>

<section class="story-column">

{#each sorted_projects as project}
  <div class="step-content">
      <h3>{project.title}</h3>
      <p>{project.story}</p>
  </div>
{/each}

</section>

<svelte:fragment slot="viz">

<div class="project-detail">

<h3 class="year">
  {sorted_projects[activeProjectIdx].year}
</h3>

<img
  src={sorted_projects[activeProjectIdx].image}
  alt={`Image for ${sorted_projects[activeProjectIdx].title}`}
/>

</div>

</svelte:fragment>

</Scrolly>

</div>

<style>

/* container */

.scrolly-wrapper {
  width: min(1100px, 94vw);
  margin: 0 auto;
}

/* story column */

.story-column {
  padding-bottom: 60vh; /* keeps last image aligned */
}

/* individual project steps */

.step-content {
  min-height: 48vh;
  padding: clamp(1rem, 2vw, 1.75rem);
  border-left: 4px solid var(--accent-color);
  margin-bottom: 1rem;
}

/* typography */

.step-content h3 {
  font-size: clamp(1.25rem, 2vw, 1.5rem);
  margin-bottom: 0.4rem;
}

.step-content p {
  font-size: clamp(0.95rem, 1.2vw, 1.05rem);
  line-height: 1.6;
  max-width: 55ch;
  margin: 0.4rem 0;
}

/* visualization panel */

.project-detail {
  padding: clamp(1rem, 2vw, 2rem);
  width: 100%;
  max-width: 650px;
  margin: 0 auto;
  text-align: center;
}

.year {
  font-size: clamp(1.4rem, 2vw, 1.9rem);
  opacity: 0.6;
  margin-bottom: 0.8rem;
}

/* image */

.project-detail img {
  width: 100%;
  height: auto;
  object-fit: contain;
  border-radius: 10px;
  box-shadow: 0 8px 22px rgba(0,0,0,0.12);
}

</style>