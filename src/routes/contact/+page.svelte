<svelte:head>
  <title>Contact me</title>
</svelte:head>
<nav> 
        <a class="home" href="../">Home</a>
        <a href="../projects">Projects</a>
        <a class="current" href="../contact">Contact</a>
        <a href="../resume">Resume</a>
        <a href="https://github.com/kveni12" target="_blank">Github</a>
    </nav>
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