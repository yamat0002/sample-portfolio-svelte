<script>
  import projects from "$lib/projects.json";
  import Project from "$lib/Project.svelte";
  import reading from "$lib/reading.json";
  import ReadingItem from "$lib/ReadingItem.svelte";
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