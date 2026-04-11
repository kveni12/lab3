<script>
  import Scrolly from "svelte-scrolly";

  export let projects = [];

  let scrollyProgress = 0;
  let activeProjectIdx = 0;

  $: sorted_projects = [...projects].sort((a, b) => a.year - b.year);

  $: {
    if (!sorted_projects.length) {
      activeProjectIdx = 0;
    } else {
      const maxIndex = sorted_projects.length - 1;
      const rawIndex = Math.floor((scrollyProgress / 100) * sorted_projects.length);
      activeProjectIdx = Math.max(0, Math.min(maxIndex, rawIndex));
    }
  }

  $: activeProject = sorted_projects[activeProjectIdx] ?? null;
</script>

<div class="scrolly-wrapper">
  <Scrolly bind:progress={scrollyProgress}>
    <section class="story-column">
      {#each sorted_projects as project, i}
        <div
          class="step-content"
          class:active={i === activeProjectIdx}
          aria-current={i === activeProjectIdx ? "true" : "false"}
        >
          {#if project.link}
            <h3>
              <a href={project.link} target="_blank" rel="noopener noreferrer">
                {project.title}
              </a>
            </h3>
          {:else}
            <h3>{project.title}</h3>
          {/if}

          <p>{project.story}</p>
        </div>
      {/each}
    </section>

    <svelte:fragment slot="viz">
      <div class="project-detail">
        {#if activeProject}
          <p class="year">{activeProject.year}</p>

          <img
            src={activeProject.image}
            alt={`Image for ${activeProject.title}`}
            loading="eager"
          />

          <div class="project-meta">
            {#if activeProject.link}
              <h3 class="project-title">
                <a href={activeProject.link} target="_blank" rel="noopener noreferrer">
                  {activeProject.title}
                </a>
              </h3>
            {:else}
              <h3 class="project-title">{activeProject.title}</h3>
            {/if}

            <p class="project-description">{activeProject.description}</p>
          </div>
        {/if}
      </div>
    </svelte:fragment>
  </Scrolly>
</div>

<style>
  .scrolly-wrapper {
    width: min(1150px, 94vw);
    margin: 0 auto;
  }

  .story-column {
    padding-bottom: 60vh;
  }

  .step-content {
    min-height: 46vh;
    padding: clamp(1rem, 2vw, 1.75rem);
    padding-left: 1.25rem;
    margin-bottom: 1rem;
    border-left: 3px solid transparent;
    opacity: 0.45;
    transform: translateY(0);
    transition:
      opacity 0.25s ease,
      border-color 0.25s ease,
      transform 0.25s ease,
      background-color 0.25s ease;
    border-radius: 0.5rem;
  }

  .step-content.active {
    border-left-color: #d8c7ff;
    opacity: 1;
    transform: translateY(-2px);
    background: rgba(216, 199, 255, 0.08);
  }

  .step-content h3 {
    font-size: clamp(1.2rem, 2vw, 1.5rem);
    margin: 0 0 0.45rem 0;
    line-height: 1.2;
  }

  .step-content p {
    font-size: clamp(0.95rem, 1.2vw, 1.05rem);
    line-height: 1.65;
    max-width: 58ch;
    margin: 0;
  }

  .step-content a,
  .project-title a {
    color: inherit;
    text-decoration: underline;
    text-underline-offset: 0.14em;
  }

  .project-detail {
    width: 100%;
    max-width: 680px;
    margin: 0 auto;
    padding: clamp(1rem, 2vw, 2rem);
    text-align: center;
  }

  .year {
    font-size: clamp(1.1rem, 1.7vw, 1.5rem);
    letter-spacing: 0.04em;
    text-transform: uppercase;
    opacity: 0.6;
    margin: 0 0 0.8rem 0;
  }

  .project-detail img {
    display: block;
    width: 100%;
    height: auto;
    object-fit: contain;
    border-radius: 14px;
    box-shadow: 0 10px 28px rgba(0, 0, 0, 0.12);
    margin-bottom: 1rem;
  }

  .project-meta {
    max-width: 60ch;
    margin: 0 auto;
  }

  .project-title {
    font-size: clamp(1.15rem, 1.8vw, 1.4rem);
    margin: 0 0 0.5rem 0;
    line-height: 1.25;
  }

  .project-description {
    font-size: clamp(0.95rem, 1.15vw, 1.02rem);
    line-height: 1.65;
    margin: 0;
    opacity: 0.9;
  }

  @media (max-width: 800px) {
    .step-content {
      min-height: 36vh;
      opacity: 1;
    }

    .project-detail {
      padding-top: 0.5rem;
    }
  }
</style>