<script>
    export let data = {};
</script>

<article>
    <div class="text">
        <h2>{data.title}</h2>
        <p class="author">by {data.author}</p>
    </div>
    <img src={data.image} alt={data.title}>
</article>

<style>
/* Book card */
article {
    display: flex;
    align-items: flex-start;
    gap: 1.5em;           /* space between text and image */
    padding: 1em;
    border: 1px solid #ddd;
    border-radius: 8px;
    background-color: #fafafa;
    flex: 0 1 320px;      /* min width for wrapping */
    transition: transform 0.2s;
}

/* Text section: title above author */
.text {
    display: flex;
    flex-direction: column;
    gap: 0.3em;
    flex: 1;              /* take remaining horizontal space */
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

/* Image on the right, next to text */
img {
    width: 100px;
    height: auto;
    border-radius: 4px;
    flex-shrink: 0;        /* don’t shrink */
}

/* Hover effect for polish */
article:hover {
    background-color: #f0f0f0;
    box-shadow: 0 2px 8px rgba(0,0,0,0.1);
    transform: translateY(-2px);
}

/* Parent container: horizontal flow with wrapping */
.readings {
    display: flex;
    flex-wrap: wrap;
    gap: 2em;             /* space between cards before wrapping */
}
</style>