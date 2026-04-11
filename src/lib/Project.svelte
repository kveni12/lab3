<script>
    export let data = {};
</script>

<article>
    <h2>{data.title}</h2>
    <img src={data.image} alt={data.title} />
    <p>
        <strong>{data.year}</strong> | {data.description}
        {#if data.link}
            <br />
            <a href={data.link} target="_blank" rel="noopener noreferrer">View Project</a>
        {/if}
    </p>
</article>

<style>
article {
  display: grid;
  grid-template-rows: subgrid;
  grid-row: span 3;
}

a {
  color: inherit;
  text-decoration: underline;
}
</style>