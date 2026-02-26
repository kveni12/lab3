<script>
import {base} from "$app/paths";
import {page} from "$app/stores";
let pages = [
  {url: "/", title: "Home"},
  {url: "/projects", title: "Projects"},
  {url: "/resume", title: "Resume"},
  {url: "/contact", title: "Contact"},
  {url: "https://github.com/kmorhun", title: "Github"},
];
</script>

<nav>   
    {#each pages as p}
        <a href={base + p.url}
            class:current={p.url === "/" // is this link the home page?
            ? $page.url.pathname === (base + "/") // if yes - set current = true if the path name matches. Else, set current = true if the path name starts correctly
            : $page.url.pathname.startsWith(base + p.url)}
            target={p.url.startsWith("http") ? "_blank": null}
            >
            {p.title}
        </a>
    {/each}
</nav>
<slot />

<style>
    nav ul,
nav li {
  display: contents;
}
nav {
  display: flex;
  margin-bottom: 1.5em;
  border-bottom: 1px solid oklch(77.65% 0.03264 291.562);
}
nav a {
  flex: 1;
  text-decoration: none;
  color: inherit;
  text-align: center;
  padding: 0.5em 0.5em 0.6em;
  margin-bottom: 2px;
}

nav a.current {
  border-bottom: 0.4em solid oklch(77.65% 0.03264 291.562);
}

nav a:hover {
  border-bottom: 0.6em solid var(--color-accent);
  background-color: oklch(from var(--color-accent) 95% 5% h);
  padding-bottom: 0.1em;
}

:root {
  --color-accent: oklch(70% 35% 300); /* light purple */
}

</style>