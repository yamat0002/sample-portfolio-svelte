<script>
  import projects from "$lib/projects.json";
  import Project from "$lib/Project.svelte";
  import ProjectNarrative from "$lib/ProjectNarrative.svelte";

  let years = projects.map(proj => proj.year);
  let range = Math.max(...years) - Math.min(...years);
</script>

<svelte:head>
  <title>Projects</title>
</svelte:head>

<h1>{projects.length} Projects over {range} Years</h1>
<p>Scroll down to see a timeline of my projects and how they've contributed to my professional and personal life</p>

<ProjectNarrative />

<div class="projects">
  {#each projects as p}
    <Project data={p} />
  {/each}
</div>

<p class="outro">Thanks for scrolling through my project story!!!</p>

<style>
  .outro {
    margin-bottom: 2rem;
  }
</style>