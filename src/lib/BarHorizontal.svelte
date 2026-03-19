<script>
  import * as d3 from "d3";

  export let data = [];

  let width = 500;
  let height = 300;
  let margin = { top: 40, right: 150, bottom: 50, left: 100 };
  let innerWidth = width - margin.left - margin.right;
  let innerHeight = height - margin.top - margin.bottom;

  let xAxis, yAxis;

  $: yScale = d3
    .scaleBand()
    .domain(data.map((d) => d.label))
    .range([0, innerHeight])
    .padding(0.4);

  $: xScale = d3
    .scaleLinear()
    .domain([0, d3.max(data, (d) => d.value) || 1])
    .range([0, innerWidth]);

  $: colorScale = d3
    .scaleOrdinal(d3.schemePastel1)
    .domain(data.map((d) => d.label));

  $: maxBar = d3.greatest(data, (d) => d.value);

  $: if (xAxis && yAxis) {
    d3.select(xAxis).call(d3.axisBottom(xScale));
    d3.select(yAxis).call(d3.axisLeft(yScale));
  }
</script>

<div class="container">
  <svg viewBox="0 0 {width} {height}">
    <text
      x={margin.left + innerWidth / 2}
      y={margin.top / 2}
      text-anchor="middle"
      class="chart-title"
    >
      Lines of Code per Language
    </text>
    <g transform="translate({margin.left}, {margin.top})">
      {#each data as d}
        <rect
          x={0}
          y={yScale(d.label)}
          width={xScale(d.value)}
          height={yScale.bandwidth()}
          fill={colorScale(d.label)}
        />
      {/each}

      {#if maxBar}
        <rect
          x={0}
          y={yScale(maxBar.label)}
          width={xScale(maxBar.value)}
          height={yScale.bandwidth()}
          fill="none"
          stroke="currentColor"
          stroke-width="2"
        />
        <line
          x1={xScale(maxBar.value) / 2}
          y1={yScale(maxBar.label)}
          x2={xScale(maxBar.value) / 2}
          y2={yScale(maxBar.label) - 20}
          stroke="currentColor"
          stroke-width="1"
        />
        <text
          x={xScale(maxBar.value) / 2}
          y={yScale(maxBar.label) - 25}
          text-anchor="middle"
          class="annotation"
        >
          Most lines of code
        </text>
      {/if}

      <text
        x={innerWidth / 2}
        y={innerHeight + margin.bottom - 5}
        text-anchor="middle"
        class="axis-label"
      >
        Lines of Code
      </text>
      <text
        x={-(innerHeight / 2)}
        y={-margin.left + 50}
        text-anchor="middle"
        transform="rotate(-90)"
        class="axis-label"
      >
        Language
      </text>
    </g>
    <g
      transform="translate({margin.left}, {margin.top + innerHeight})"
      bind:this={xAxis}
    />
    <g transform="translate({margin.left}, {margin.top})" bind:this={yAxis} />
  </svg>
  <ul class="legend">
    {#each data as d}
      <li style="--color: {colorScale(d.label)}">
        <span class="swatch"></span>
        {d.label} <em>({d.value})</em>
      </li>
    {/each}
  </ul>
</div>

<style>
  svg {
    max-width: 100%;
    height: auto;
    overflow: visible;
  }

  .container {
    display: flex;
  }

  .legend {
    flex: 1;
    list-style: none;
    padding: 0;
    margin: 0;
  }

  .legend li {
    display: flex;
    align-items: center;
    gap: 0.3em;
    margin-bottom: 0.4em;
  }

  .swatch {
    display: inline-block;
    width: 12px;
    height: 12px;
    background-color: var(--color);
  }

  .chart-title {
    font-size: 1em;
    font-weight: bold;
    fill: currentColor;
  }

  .axis-label {
    font-size: 0.8em;
    fill: currentColor;
  }

  .annotation {
    font-size: 0.7em;
    fill: black;
    font-style: italic;
  }
</style>
