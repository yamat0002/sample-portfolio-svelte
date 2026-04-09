<script>
  import * as d3 from "d3";

  export let data = [];

  let width = 1000;
  let height = 300;
  let margin = { top: 20, right: 20, bottom: 40, left: 60 };

  let usableArea = {
    top: margin.top,
    right: width - margin.right,
    bottom: height - margin.bottom,
    left: margin.left,
  };
  usableArea.width = usableArea.right - usableArea.left;
  usableArea.height = usableArea.bottom - usableArea.top;

  let xAxis, yAxis;
  let hoveredDay = null;

  $: xScale = d3.scaleTime()
    .domain(d3.extent(data, d => d.date) || [0, 1])
    .range([usableArea.left, usableArea.right]);

  $: yScale = d3.scaleLinear()
    .domain([0, d3.max(data, d => d.count) || 1])
    .range([usableArea.bottom, usableArea.top])
    .nice();

  $: line = d3.line()
    .x(d => xScale(d.date))
    .y(d => yScale(d.count))
    .curve(d3.curveBumpX);

  $: {
    if (xAxis) d3.select(xAxis).call(d3.axisBottom(xScale));
    if (yAxis) d3.select(yAxis).call(d3.axisLeft(yScale));
  }

  $: dayRegions = (() => {
    if (data.length === 0) return [];
    return data.map((d, i, arr) => {
      const prev = arr[i - 1];
      const next = arr[i + 1];
      const left = prev ? new Date((d.date.getTime() + prev.date.getTime()) / 2) : d.date;
      const right = next ? new Date((d.date.getTime() + next.date.getTime()) / 2) : d.date;
      return {
        date: d.date,
        weekday: d.date.toLocaleString("en", { weekday: "long" }),
        x: xScale(left),
        width: xScale(right) - xScale(left),
      };
    });
  })();

  $: titleText = hoveredDay
    ? `Lines Edited on ${hoveredDay}s`
    : "Lines Edited by Day";
</script>

<h3 class="line-title">{titleText}</h3>
<svg viewBox="0 0 {width} {height}" on:mouseleave={() => hoveredDay = null}>
  <g transform="translate(0, {usableArea.bottom})" bind:this={xAxis} />
  <g transform="translate({usableArea.left}, 0)" bind:this={yAxis} />

  {#each dayRegions as region}
    <rect
      x={region.x}
      y={usableArea.top}
      width={region.width}
      height={usableArea.bottom - usableArea.top}
      fill="transparent"
      on:mouseenter={() => hoveredDay = region.weekday}
    />
  {/each}

  {#each dayRegions as region}
    {#if region.weekday === hoveredDay}
      <rect
        x={region.x}
        y={usableArea.top}
        width={region.width}
        height={usableArea.bottom - usableArea.top}
        fill="var(--color-accent)"
        opacity="0.2"
      />
    {/if}
  {/each}

  <path
    d={line(data)}
    fill="none"
    stroke="steelblue"
    stroke-width="2"
  />

  {#each data as d}
    {@const isHighlighted = d.date.toLocaleString("en", { weekday: "long" }) === hoveredDay}
    <circle
      cx={xScale(d.date)}
      cy={yScale(d.count)}
      r={isHighlighted ? 5 : 3}
      fill={isHighlighted ? "var(--color-accent)" : "steelblue"}
    />
    {#if isHighlighted}
      <text
        x={xScale(d.date)}
        y={usableArea.top + 15}
        text-anchor="middle"
        font-size="12"
        fill="var(--color-accent)"
      >
        {Math.round(d.count)}
      </text>
    {/if}
  {/each}

  <text
    x={usableArea.left + usableArea.width / 2}
    y={height - 5}
    text-anchor="middle"
    class="axis-label">
    Date
  </text>
  <text
    x={-(usableArea.top + usableArea.height / 2)}
    y={10}
    text-anchor="middle"
    transform="rotate(-90)"
    class="axis-label">
    Number of Lines Edited
  </text>
</svg>

<style>
  svg {
    width: 100%;
    max-height: 30vh;
    overflow: visible;
  }

  .line-title {
    text-align: center;
    margin: 0.3em 0 0.1em;
    font-size: 0.95em;
  }

  .axis-label {
    font-size: 0.8em;
    fill: currentColor;
  }
</style>