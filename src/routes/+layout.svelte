<script>
  import { base } from "$app/paths";
  import { page } from "$app/stores";
  import { browser } from "$app/environment";

  let pages = [
    { url: "/", title: "About" },
    { url: "/projects", title: "Projects" },
    { url: "/resume", title: "Resume" },
    { url: "/contact", title: "Contact" },
    { url: "/meta", title: "Meta" },
    { url: "https://github.com/yamat0002", title: "Github" },
  ];

  let root = globalThis.document?.documentElement;
  let localStorage = globalThis.localStorage ?? {};

  let colorScheme = localStorage.colorScheme ?? "light dark";

  $: root?.style.setProperty("color-scheme", colorScheme);
  $: if (browser) {
    localStorage.colorScheme = colorScheme;
  }
</script>

<label class="color-scheme-switch">
  Theme:
  <select bind:value={colorScheme}>
    <option value="light dark">Automatic</option>
    <option value="light">Light</option>
    <option value="dark">Dark</option>
  </select>
</label>

<nav>
  {#each pages as p}
    <a
      href={p.url.startsWith("http") ? p.url : base + p.url}
      class:current={p.url === "/"
        ? $page.url.pathname === base + "/"
        : $page.url.pathname.startsWith(base + p.url)}
      target={p.url.startsWith("http") ? "_blank" : null}
    >
      {p.title}
    </a>
  {/each}
</nav>

<slot />

<style>
  .color-scheme-switch {
    position: absolute;
    top: 1rem;
    right: 1rem;
    display: inline-flex;
    gap: 4px;
    font-size: 80%;
  }

  nav {
    --border-color: oklch(50% 10% 200 / 40%);
    display: flex;
    margin-bottom: 1em;
    border-bottom: 2px solid var(--border-color);
  }

  nav a {
    flex: 1;
    text-decoration: none;
    color: inherit;
    text-align: center;
    padding: 0.5em;
  }

  nav a.current {
    border-bottom: 4px solid var(--border-color);
    padding-bottom: 0.1em;
  }

  nav a:hover {
    border-bottom: 4px solid var(--color-accent);
    background-color: color-mix(in oklch, var(--color-accent), canvas 85%);
  }
</style>