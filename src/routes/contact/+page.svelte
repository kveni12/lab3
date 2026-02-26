<svelte:head>
  <title>Contact me</title>
</svelte:head>
    <h1>Contact Me</h1>
    <form action="mailto:kveni@mit.edu" method="GET">
        <label>
            Email:
            <input name="Email" type="email">
        </label>

        <label>
            Subject:
            <input name="subject">
        </label>

        <label>
            Message:
            <textarea name="body" rows="10" cols="30"></textarea>
        </label>

        <button type="submit">Submit</button>
    </form>