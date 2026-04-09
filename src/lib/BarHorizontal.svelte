<script>
  import * as d3 from "d3";

  export let data = [];
  export let title = "";

  let width = 600;
  let height = 180;
  let margin = { top: 5, right: 100, bottom: 35, left: 70 };
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
    let maxVal = d3.max(data, (d) => d.value) || 1;
    d3.select(xAxis).call(
      d3.axisBottom(xScale)
        .ticks(Math.min(maxVal, 10))
        .tickFormat(d => Number.isInteger(d) ? d : "")
    );
    d3.select(yAxis).call(d3.axisLeft(yScale));
  }
</script>

<h3 class="chart-title">{title}</h3>
<div class="container">
  <svg viewBox="0 0 {width} {height}">
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
        <text
          x={xScale(maxBar.value) + 5}
          y={yScale(maxBar.label) + yScale.bandwidth() / 2}
          dominant-baseline="middle"
          text-anchor="start"
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
        Number of Lines
      </text>
      <text
        x={-(innerHeight / 2)}
        y={-margin.left + 20}
        text-anchor="middle"
        transform="rotate(-90)"
        class="axis-label"
      >
        Programming Language
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
    flex: 3;
  }

  .container {
    display: flex;
    align-items: start;
    gap: 1em;
  }

  .legend {
    list-style: none;
    padding: 0;
    margin: 0;
    font-size: 0.75em;
    flex: 1;
    min-width: 8em;
  }

  .legend li {
    display: flex;
    align-items: center;
    gap: 0.3em;
    margin-bottom: 0.3em;
  }

  .swatch {
    display: inline-block;
    width: 10px;
    height: 10px;
    background-color: var(--color);
    flex-shrink: 0;
  }

  .chart-title {
    font-size: 0.95em;
    text-align: center;
    margin: 0.3em 0 0.1em;
  }

  .axis-label {
    font-size: 0.6em;
    fill: currentColor;
  }

  .annotation {
    font-size: 0.5em;
    fill: currentColor;
    font-style: italic;
  }
</style>