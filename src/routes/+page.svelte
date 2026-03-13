<script>
  import { onMount } from "svelte";
  import projects from "$lib/projects.json";
  import Project from "$lib/Project.svelte";
  import reading from "$lib/reading.json";
  import ReadingItem from "$lib/ReadingItem.svelte";
  let githubData = null;
  let loading = true;
  let error = null;
  onMount(async () => {
    try {
      console.log("Page has been mounted!")
      let response = await fetch("https://api.github.com/users/yamat0002");
      console.log(response);
      githubData = await response.json();
      console.log(githubData);
    } catch (err) {
      error = err;
    }
    loading = false;
  });
</script>

<h1>Ayat Abodayeh</h1>
<img
  src="images/ayat.jpg"
  class="profile-photo"
  alt="A cartoonish photo of me but in a beanie and done in picrew"
/>
<p>
  I am an MAS Master's student at LLK and I enjoy being part of creative
  learning spaces and working towards greater social justice ~~~
</p>

{#if loading}
    <p>Loading...</p>
{:else if error}
    <p>Something went wrong: {error.message}</p>
{:else}
    <section class="github-stats">
        <h2>My GitHub Stats</h2>
        <dl>
            <dt>Followers</dt>
            <dd>{githubData.followers}</dd>
            <dt>Following</dt>
            <dd>{githubData.following}</dd>
            <dt>Public Repositories</dt>
            <dd>{githubData.public_repos}</dd>
            <dt>Public Gists</dt>
            <dd>{githubData.public_gists}</dd>
        </dl>
    </section>
{/if}

<h2>Featured Projects</h2>
<div class="projects">
  {#each projects.slice(0, 3) as p}
    <Project data={p} />
  {/each}
</div>

<h2>What I'm Reading</h2>
<div class="reading">
  {#each reading as r}
    <ReadingItem data={r} />
  {/each}
</div>

<style>
    .github-stats dl {
        display: grid;
        grid-template-columns: repeat(4, 1fr);
        gap: 0.5em;
        text-align: center;
        padding: 1em;
        border-left: 0.3em solid var(--color-accent);
    }

    .github-stats dt {
        grid-row: 1;
        font-weight: bold;
        color: gray;
        font-size: 0.9em;
    }

    .github-stats dd {
        grid-row: 2;
        margin: 0;
        font-size: 1.5em;
        font-weight: bold;
    }
</style>