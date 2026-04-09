<script>
  import { base } from '$app/paths';
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  import BarHorizontal from '$lib/BarHorizontal.svelte';
  import LineChart from '$lib/LineChart.svelte';
  import {
    computePosition,
    autoPlacement,
    offset,
  } from '@floating-ui/dom';

  let locData = [];
  let commits = [];
  let clickedCommits = [];

  let width = 1000;
  let height = 300;
  let margin = { top: 10, right: 10, bottom: 30, left: 60 };

  let usableArea = {
    top: margin.top,
    right: width - margin.right,
    bottom: height - margin.bottom,
    left: margin.left,
  };
  usableArea.width = usableArea.right - usableArea.left;
  usableArea.height = usableArea.bottom - usableArea.top;

  let xAxis, yAxis, yAxisGridlines;
  let svg;

  let hoveredIndex = -1;
  $: hoveredCommit = commits[hoveredIndex] ?? hoveredCommit ?? {};
  let commitTooltip;
  let tooltipPosition = { x: 0, y: 0 };

  $: brushSelection = null;

  function brushed(evt) {
    brushSelection = evt.selection;
  }

  function isCommitBrushed(commit) {
    if (!brushSelection) return false;
    let [[x0, y0], [x1, y1]] = brushSelection;
    let cx = xScale(commit.datetime);
    let cy = yScale(commit.hourFrac);
    return cx >= x0 && cx <= x1 && cy >= y0 && cy <= y1;
  }

  $: brushedCommits = brushSelection ? commits.filter(isCommitBrushed) : [];
  $: selectedCommits = Array.from(new Set([...clickedCommits, ...brushedCommits]));

  let linesByDate = [];

  onMount(async () => {
    locData = await d3.csv(`${base}/loc.csv`, row => ({
      ...row,
      line: Number(row.line),
      length: Number(row.length),
      depth: Number(row.depth),
      date: new Date(row.date + "T00:00" + row.timezone),
      datetime: new Date(row.datetime),
    }));

    commits = d3.groups(locData, d => d.commit).map(([commit, lines]) => {
      let first = lines[0];
      let { author, date, time, timezone, datetime } = first;
      return {
        id: commit,
        url: "https://github.com/yamat0002/sample-portfolio-svelte/commit/" + commit,
        author, date, time, timezone, datetime,
        hourFrac: datetime.getHours() + datetime.getMinutes() / 60,
        totalLines: lines.length,
        lines: lines,
      };
    });

    commits = d3.sort(commits, d => -d.totalLines);
  });

  $: {
    let rolled = d3.rollups(locData, v => v.length, d => d3.timeDay.floor(d.datetime))
      .map(([date, count]) => ({ date, count }));
    let [minD, maxD] = d3.extent(rolled, d => d.date) || [new Date(), new Date()];
    let allDays = d3.timeDays(minD, d3.timeDay.offset(maxD, 1));
    linesByDate = allDays.map(date => ({
      date,
      count: rolled.find(r => r.date.getTime() === date.getTime())?.count ?? 0,
    }));
  }

  $: minDate = d3.min(commits, d => d.datetime);
  $: maxDate = d3.max(commits, d => d.datetime);
  $: maxDatePlusOne = maxDate ? d3.timeDay.offset(maxDate, 1) : undefined;

  $: xScale = d3.scaleTime()
    .domain(minDate && maxDatePlusOne ? [minDate, maxDatePlusOne] : [0, 1])
    .range([usableArea.left, usableArea.right])
    .nice();

  $: yScale = d3.scaleLinear()
    .domain([0, 24])
    .range([usableArea.bottom, usableArea.top]);

  $: rScale = d3.scaleSqrt()
    .domain(d3.extent(commits, d => d.totalLines) || [0, 1])
    .range([4, 20]);

  $: {
    if (xAxis) {
      d3.select(xAxis).call(d3.axisBottom(xScale));
    }
    if (yAxis) {
      d3.select(yAxis).call(
        d3.axisLeft(yScale)
          .tickFormat(d => String(d % 24).padStart(2, "0") + ":00")
      );
    }
    if (yAxisGridlines) {
      d3.select(yAxisGridlines).call(
        d3.axisLeft(yScale)
          .tickFormat("")
          .tickSize(-usableArea.width)
      );
    }
  }

  $: {
    if (svg) {
      d3.select(svg).call(d3.brush()
        .extent([[usableArea.left, usableArea.top], [usableArea.right, usableArea.bottom]])
        .on("start brush end", brushed)
      );
      d3.select(svg).selectAll(".dots, .overlay ~ *").raise();
    }
  }

  async function dotInteraction(index, evt) {
    let hoveredDot = evt.target;
    if (evt.type === "mouseenter") {
      hoveredIndex = index;
      tooltipPosition = await computePosition(hoveredDot, commitTooltip, {
        strategy: "fixed",
        middleware: [offset(5), autoPlacement()],
      });
    } else if (evt.type === "mouseleave") {
      hoveredIndex = -1;
    } else if (evt.type === "click") {
      let commit = commits[index];
      if (!clickedCommits.includes(commit)) {
        clickedCommits = [...clickedCommits, commit];
      } else {
        clickedCommits = clickedCommits.filter(c => c !== commit);
      }
    }
  }

  $: selectedLines = (selectedCommits.length > 0 ? selectedCommits : commits)
    .flatMap(d => d.lines);

  $: languageCounts = d3.rollup(selectedLines, v => v.length, d => d.type);

  $: allLanguages = Array.from(new Set(locData.map(d => d.type)));

  $: barData = allLanguages.map(lang => ({
    label: lang,
    value: languageCounts.get(lang) ?? 0,
  }));

  $: barTitle = selectedCommits.length > 0
    ? `Lines of Code: ${selectedCommits.length} Selected Commits`
    : "Lines of Code: Website Breakdown";
</script>

<svelte:head>
  <title>Meta</title>
</svelte:head>

<h1>Meta</h1>
<p>This page includes stats about the code of this website.</p>

<h2>Commits by time of day</h2>

<svg viewBox="0 0 {width} {height}" class="scatterplot" bind:this={svg}>
  <g class="gridlines" transform="translate({usableArea.left}, 0)"
     bind:this={yAxisGridlines} />
  <g transform="translate(0, {usableArea.bottom})" bind:this={xAxis} />
  <g transform="translate({usableArea.left}, 0)" bind:this={yAxis} />
  <g class="dots">
    {#each commits as commit, index}
      <circle
        cx={xScale(commit.datetime)}
        cy={yScale(commit.hourFrac)}
        r={rScale(commit.totalLines)}
        fill="steelblue"
        fill-opacity="0.6"
        class:selected={selectedCommits.includes(commit)}
        on:mouseenter={evt => dotInteraction(index, evt)}
        on:mouseleave={evt => dotInteraction(index, evt)}
        on:click={evt => dotInteraction(index, evt)}
      />
    {/each}
  </g>
</svg>

<dl class="info tooltip"
    hidden={hoveredIndex === -1}
    bind:this={commitTooltip}
    style="top: {tooltipPosition.y}px; left: {tooltipPosition.x}px;">
  <dt>Commit</dt>
  <dd><a href="{hoveredCommit.url}" target="_blank">{hoveredCommit.id}</a></dd>
  <dt>Author</dt>
  <dd>{hoveredCommit.author}</dd>
  <dt>Date</dt>
  <dd>{hoveredCommit.datetime?.toLocaleString("en", { dateStyle: "full" })}</dd>
  <dt>Time</dt>
  <dd>{hoveredCommit.datetime?.toLocaleString("en", { timeStyle: "short" })}</dd>
  <dt>Lines</dt>
  <dd>{hoveredCommit.totalLines}</dd>
</dl>

<BarHorizontal data={barData} title={barTitle} />

<LineChart data={linesByDate} />

<style>
  h1, h2, p {
    margin: 0.2em 0;
  }

  .scatterplot {
    width: 100%;
    max-height: 35vh;
    overflow: visible;
  }

  .gridlines {
    stroke-opacity: 0.2;
  }

  circle {
    fill-opacity: 0.6;
    transition: 200ms;
    cursor: pointer;

    &:hover {
      fill: orange;
    }
  }

  .selected {
    fill: var(--color-accent);
  }

  @keyframes marching-ants {
    to {
      stroke-dashoffset: -8;
    }
  }

  svg :global(.selection) {
    fill-opacity: 10%;
    stroke: black;
    stroke-opacity: 70%;
    stroke-dasharray: 5 3;
    animation: marching-ants 2s linear infinite;
  }

  dl.info {
    display: grid;
    grid-template-columns: auto 1fr;
    gap: 0.2em 0.8em;
    margin: 0;
    padding: 0.6em 0.8em;
    font-size: 0.85em;
    background-color: oklch(100% 0% 0 / 85%);
    backdrop-filter: blur(8px);
    border-radius: 8px;
    box-shadow: 0 4px 16px oklch(0% 0% 0 / 15%);
    transition-duration: 500ms;
    transition-property: opacity, visibility;

    &[hidden]:not(:hover, :focus-within) {
      opacity: 0;
      visibility: hidden;
    }
  }

  dl.info dt {
    font-weight: bold;
    text-transform: uppercase;
    font-size: 0.75em;
    color: gray;
    align-self: center;
  }

  dl.info dd {
    margin: 0;
  }

  .tooltip {
    position: fixed;
    z-index: 10;
  }
</style>