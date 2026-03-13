<script>
  import projects from "$lib/projects.json";
  import Project from "$lib/Project.svelte";
  import ProjectNarrative from "$lib/ProjectNarrative.svelte";

</script>

<svelte:head>
  <title>Projects</title>
  </svelte:head>


    <h1>{projects.length} Projects</h1>
<p>Scroll down to see my a timeline of my projects and how they've contributed to my professional and personal life</p>
   <ProjectNarrative />

<div class="projects">
  
  {#each projects as p}
  <Project data={p} />

  {/each}
</div>

<p class="outro">Thanks for scrolling through my project story! Feel free to explore all of the projects at your leisure below.</p>


<style>
  .outro{
    margin-bottom: 2rem;
  }
</style>

