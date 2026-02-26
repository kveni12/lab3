<script>
import {base} from "$app/paths";
import {page} from "$app/stores";
let colorScheme = "light dark";
let localStorage = globalThis.localStorage ?? {};
if (localStorage.colorScheme) { // if localStorage has a colorScheme property
  colorScheme = localStorage.colorScheme; // override the default colorScheme
}
let root = globalThis.document?.documentElement;
$: root?.style.setProperty("color-scheme", colorScheme);
$: localStorage.colorScheme = colorScheme; // persist the user's choice in localStorage
let pages = [
  {url: "/", title: "Home"},
  {url: "/projects", title: "Projects"},
  {url: "/resume", title: "Resume"},
  {url: "/contact", title: "Contact"},
  {url: "https://github.com/kveni12", title: "Github"},
];
</script>
<label class="color-scheme-switch">
    Theme:
    <select bind:value = {colorScheme}>
        <option value="light dark">Automatic</option>
        <option value="light">Light</option>
        <option value="dark">Dark</option>
    </select>
</label>
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

nav {
    --border-color: oklch(50.188% 0.14837 285.869 / 40%);
    border-bottom: 2px solid var(--border-color);
}
nav a.current {
  border-bottom: 0.4em solid var(--border-color);
}

nav a:hover {
  border-bottom: 0.6em solid var(--color-accent);
  background-color: color-mix(in oklch, var(--color-accent), canvas 85%);

  padding-bottom: 0.1em;
}

:root {
  --color-accent: oklch(70% 35% 300); /* light purple */
}
.color-scheme-switch {
  position: absolute;
  top: 1rem;
  right: 1rem;

  display: inline-flex;
  gap: 4px;

  font-size: 80%;
  font-family: inherit;
}
</style>