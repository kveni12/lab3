<script>
  import * as d3 from "d3";

  export let data = [];

  let width = 500;
  let height = 350;

  let margin = { top: 40, right: 150, bottom: 50, left: 70 };

  let innerWidth = width - margin.left - margin.right;
  let innerHeight = height - margin.top - margin.bottom;

  let selectedIndex = -1;

  // accessibility state
  let liveText = "";
  let showChart = true;

  function toggleBar(index, event) {
    if (!event.key || event.key === "Enter" || event.key === " ") {
      selectedIndex = index;

      const d = data[index];
      if (d) {
        liveText = `${d.label}: ${d.value} projects selected.`;
      }
    }
  }

  function toggleView() {
    showChart = !showChart;
    liveText = showChart
      ? "Bar chart view shown."
      : "Table view shown.";
  }

  // scales
  $: xScale = d3.scaleBand()
    .domain(data.map(d => d.label))
    .range([0, innerWidth])
    .padding(0.25);

  $: maxVal = d3.max(data, d => d.value) || 1;

  $: yScale = d3.scaleLinear()
    .domain([0, maxVal + 1])
    .range([innerHeight, 0])
    .nice();

  $: colorScale = d3.scaleOrdinal()
  .domain(data.map(d => d.label))
  .range(d3.quantize(d3.interpolateBlues, data.length));

  $: maxBar = d3.greatest(data, d => d.value);

  // dynamic description
  $: description = `A bar chart showing project counts by year. ${
    data.map(d => `${d.label}: ${d.value} projects`).join(", ")
  }.`;

  let xAxis;
  let yAxis;

  $: if (xAxis && yAxis) {
    d3.select(xAxis).call(d3.axisBottom(xScale));

    d3.select(yAxis).call(
      d3.axisLeft(yScale)
        .ticks(maxVal)
        .tickFormat(d => Number.isInteger(d) ? d : "")
    );
  }
</script>

<!-- toggle button -->
<button
  on:click={toggleView}
  aria-pressed={!showChart}
  aria-label="Toggle between bar chart and table view"
  class="toggle-button"
>
  {showChart ? "Show Table" : "Show Chart"}
</button>

{#if showChart}
  <div class="container">
    <svg
      viewBox={`0 0 ${width} ${height}`}
      role="img"
      aria-labelledby="bar-title bar-desc"
      tabindex="0"
    >
      <title id="bar-title">Projects by Year</title>
      <desc id="bar-desc">{description}</desc>

      <text
        x={width / 2}
        y="25"
        text-anchor="middle"
        class="chart-title"
      >
        Projects per Year
      </text>

      <g transform={`translate(${margin.left},${margin.top})`}>
        <g bind:this={yAxis}></g>

        <text
          transform="rotate(-90)"
          x={-innerHeight / 2}
          y={-50}
          text-anchor="middle"
          class="axis-label"
        >
          Number of Projects
        </text>

        {#each data as d, index}
          <rect
            x={xScale(d.label)}
            y={yScale(d.value)}
            width={xScale.bandwidth()}
            height={innerHeight - yScale(d.value)}
            fill={colorScale(d.label)}
            stroke="black"
            opacity={selectedIndex === -1 || selectedIndex === index ? 1 : 0.45}
            tabindex="0"
            role="button"
            aria-label={`Year ${d.label}, ${d.value} projects`}
            aria-pressed={selectedIndex === index}
            on:click={(e) => toggleBar(index, e)}
            on:keyup={(e) => toggleBar(index, e)}
          />
        {/each}

        {#if maxBar}
          <rect
            x={xScale(maxBar.label)}
            y={yScale(maxBar.value)}
            width={xScale.bandwidth()}
            height={innerHeight - yScale(maxBar.value)}
            fill="none"
            stroke="currentColor"
            stroke-width="2"
          />

          <line
            x1={xScale(maxBar.label) + xScale.bandwidth()}
            y1={yScale(maxBar.value) + (innerHeight - yScale(maxBar.value)) / 2}
            x2={xScale(maxBar.label) + xScale.bandwidth() + 30}
            y2={yScale(maxBar.value) + (innerHeight - yScale(maxBar.value)) / 2}
            stroke="currentColor"
          />

          <text
            x={xScale(maxBar.label) + xScale.bandwidth() + 35}
            y={yScale(maxBar.value) + (innerHeight - yScale(maxBar.value)) / 2}
            dominant-baseline="middle"
            class="annotation"
          >
            Year with most projects ({maxBar.label})
          </text>
        {/if}

        <g
          transform={`translate(0,${innerHeight})`}
          bind:this={xAxis}
        ></g>

        <text
          x={innerWidth / 2}
          y={innerHeight + 40}
          text-anchor="middle"
          class="axis-label"
        >
          Year
        </text>
      </g>
    </svg>

    <!-- live region -->
    <p aria-live="polite" class="sr-only">{liveText}</p>

    <!-- legend -->
    <ul class="legend">
      {#each data as d}
        <li style="--color: {colorScale(d.label)}">
          <span class="swatch"></span>
          {d.label} <em>({d.value})</em>
        </li>
      {/each}
    </ul>
  </div>

{:else}
  <!-- accessible table -->
  <table aria-label="Table showing project counts by year" class="data-table">
    <caption>Projects by Year</caption>
    <thead>
      <tr>
        <th scope="col">Year</th>
        <th scope="col">Projects</th>
      </tr>
    </thead>
    <tbody>
      {#each data as d, i}
        <tr>
          <th scope="row" id="row-{i}">{d.label}</th>
          <td aria-labelledby="row-{i}">{d.value}</td>
        </tr>
      {/each}
    </tbody>
  </table>

  <!-- still announce changes -->
  <p aria-live="polite" class="sr-only">{liveText}</p>
{/if}

<style>
  svg {
    max-width: 100%;
    height: auto;
  }

  .container {
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: center;
    gap: 2rem;
  }

  .legend {
    list-style: none;
    padding: 0;
    margin: 0;
    display: flex;
    flex-direction: column;
    gap: 0.6rem;
  }

  .swatch {
    width: 12px;
    height: 12px;
    background: var(--color);
    display: inline-block;
    margin-right: 6px;
  }

  li {
    display: flex;
    align-items: center;
    font-size: 13px;
  }

  .chart-title {
    font-size: 18px;
    font-weight: 600;
  }

  .axis-label {
    font-size: 13px;
    fill: #444;
  }

  .annotation {
    font-size: 11px;
    fill: black;
    font-style: italic;
  }

  rect {
    transition: 300ms;
    outline: none;
    stroke: black;
    stroke-width: 1;
  }

  svg:hover rect:not(:hover),
  .container:focus-within rect:not(:focus-visible) {
    opacity: 0.5;
  }

  rect:focus-visible {
    stroke: white;
    stroke-width: 2px;
    stroke-dasharray: 4;
  }

  .toggle-button {
    margin-bottom: 1rem;
    padding: 0.5rem 1rem;
    cursor: pointer;
  }

  .data-table {
    margin-top: 1rem;
    border-collapse: collapse;
    width: 100%;
    max-width: 30em;
  }

  .data-table caption {
    font-weight: bold;
    margin-bottom: 0.5em;
    text-align: left;
  }

  .data-table th,
  .data-table td {
    border: 1px solid #ccc;
    padding: 0.5em;
  }

  .data-table th {
    background-color: #f0f0f0;
  }

  .sr-only {
    position: absolute;
    left: -9999px;
    width: 1px;
    height: 1px;
    overflow: hidden;
  }
</style>