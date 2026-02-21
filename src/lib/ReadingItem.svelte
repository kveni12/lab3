<script>
    export let data = {};
</script>

<article>
    <h2>{data.title}</h2>
    <p class="author">by {data.author}</p>
    <img src={data.image} alt="{data.title}">
</article>

<style>
article {
    display: flex;
    align-items: flex-start;
    gap: 1em;
    padding: 1em;
    border: 1px solid #ddd;
    border-radius: 8px;
    background-color: #fafafa;
    flex: 0 1 300px; /* allow shrinking and growing, set minimum width */
}

.text {
    display: flex;
    flex-direction: column;
    gap: 0.25em;
}

h2 {
    margin: 0;
    font-size: 1.1em;
}

.author {
    margin: 0;
    font-size: 0.9em;
    color: gray;
}

img {
    width: 80px;
    height: auto;
    border-radius: 4px;
    flex-shrink: 0;
}

article:hover {
    background-color: #f0f0f0;
    box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}

.readings {
    display: flex;
    flex-wrap: wrap;
    gap: 1em;
}
</style>