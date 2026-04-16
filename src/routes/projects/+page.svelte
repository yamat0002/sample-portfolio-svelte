<script>
  import projects from "$lib/projects.json";
  import Project from "$lib/Project.svelte";
  import ProjectNarrative from "$lib/ProjectNarrative.svelte";
  import { onMount } from "svelte";
  import * as d3 from "d3";
  import Bar from '$lib/Bar.svelte';


  let rawData = [];

  let wrangled = [];

  onMount(async () => {
      rawData = await d3.json('/lab6_example.json');
      wrangled = d3.rollups(
          rawData,
          v => d3.sum(v, d => d.lines),
          d => d.language
      );
  });


  let years = projects.map((proj) => proj.year);
  let range = Math.max(...years) - Math.min(...years);

  $: barData = d3.rollups(projects, v => v.length, d => d.year)
    .map(([year, count]) => ({ label: String(year), value: count }));

</script>

<svelte:head>
  <title>Projects</title>
</svelte:head>

<h1>{projects.length} Projects over {range} Years</h1>
<Bar data={barData} />
<p><strong>Overview:</strong>
  Scroll down to see a timeline of my projects and how they've contributed to my
  professional and personal life
</p>

<ProjectNarrative />

<div class="projects">
  {#each projects as p}
    {#if p.url}
      <a href={p.url} target="_blank" class="project-link">
        <Project data={p} />
      </a>
    {:else}
      <Project data={p} />
    {/if}
  {/each}
</div>

<p class="outro">Thanks for scrolling through my project story!!!</p>


<style>
  .outro {
    margin-bottom: 2rem;
  }

 .project-link {
  text-decoration: none;
  color: inherit;
  display: contents;
}
</style>