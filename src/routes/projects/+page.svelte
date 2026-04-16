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
    <Project data={p} />
    {#if p.url}
      <a class="project-link" href={p.url} target="_blank">View Live Project →</a>
    {/if}
  {/each}
</div>

<p class="outro">Thanks for scrolling through my project story!!!</p>


<style>
  .outro {
    margin-bottom: 2rem;
  }

  .project-link {
    display: inline-block;
    margin-bottom: 1.5rem;
    color: #2d6a4f;
    font-weight: 600;
    text-decoration: none;
  }

  .project-link:hover {
    text-decoration: underline;
  }
</style> 