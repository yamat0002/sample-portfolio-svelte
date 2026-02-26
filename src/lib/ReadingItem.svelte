<script>
  export let data;
</script>

<div class="reading-item">
  <img src={data.image} alt="Cover of {data.title}" />
  <div class="reading-details">
    <h3 class="reading-title">{data.title}</h3>
    <p class="reading-author">{data.author}</p>
  </div>
</div>

<style>
  .reading-item {
    display: flex;
    gap: 1em;
    padding: 1em;
    background: oklch(from oklch(80% 3% 200) 96% 2% h);
    border-radius: 10px;
    border-left: 4px solid oklch(80% 3% 200);
    transition: transform 0.2s ease, box-shadow 0.2s ease;
  }

  .reading-item:hover {
    transform: translateY(-3px);
    box-shadow: 0 4px 12px oklch(from oklch(80% 3% 200) 70% 5% h / 0.2);
  }

  .reading-item img {
    width: 7em;
    height: 10em;
    object-fit: cover;
    border-radius: 4px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.12);
  }

  .reading-details {
    display: flex;
    flex-direction: column;
    justify-content: center;
  }

  .reading-title {
    margin: 0;
    font-size: 1.05em;
    font-weight: 600;
    line-height: 1.3;
  }

  .reading-author {
    margin: 0.4em 0 0;
    color: oklch(45% 3% 200);
    font-size: 0.88em;
  }
</style>