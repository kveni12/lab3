<script>
    export let data = {};
</script>

<article>
    <img src={data.image} alt={data.title}>
    <div class="text">
        <h2>{data.title}</h2>
        <p class="author">by {data.author}</p>
    </div>
</article>

<style>
/* Book card */
article {
    display: flex;
    align-items: flex-start;
    gap: 1.5em;           /* space between image and text */
    padding: 1em;
    border: 1px solid #ddd;
    border-radius: 8px;
    background-color: rgba(128, 128, 128, 0.1);
    flex: 0 1 320px;      /* min width for wrapping */
    transition: transform 0.2s;
}

/* Image on the left */
img {
    width: 100px;
    height: auto;
    border-radius: 4px;
    flex-shrink: 0;
}

/* Text section: title above author */
.text {
    display: flex;
    flex-direction: column;
    gap: 0.25em;
    color: black;
    flex: 1; /* takes remaining space */
}

/* Title styling */
h2 {
    margin: 0;
    font-size: 1.2em;
}

/* Author under title */
.author {
    margin: 0;
    font-size: 0.95em;
    color: gray;
}

/* Hover effect for polish */
article:hover {
    background-color: rgba(128, 128, 128, 0.2);
    box-shadow: 0 2px 8px rgba(0,0,0,0.1);
    transform: translateY(-2px);
}
</style>