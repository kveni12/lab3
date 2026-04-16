<script>
import * as d3 from "d3";

export let data = [];

let width = 500;
let height = 350;

let margin = { top: 40, right: 150, bottom: 50, left: 70 };

let innerWidth = width - margin.left - margin.right;
let innerHeight = height - margin.top - margin.bottom;

/* NEW: selection state */
let selectedIndex = -1;

/* NEW: handler */
function toggleBar(index, event) {
	if (!event.key || event.key === "Enter") {
		selectedIndex = index;
	}
}

/* x scale */
$: xScale = d3.scaleBand()
    .domain(data.map(d => d.label))
    .range([0, innerWidth])
    .padding(0.25);

/* max value */
$: maxVal = d3.max(data, d => d.value) || 1;

/* y scale */
$: yScale = d3.scaleLinear()
    .domain([0, maxVal + 1])
    .range([innerHeight, 0])
    .nice();

/* color scale */
$: colorScale = d3.scaleOrdinal(d3.schemeTableau10)
    .domain(data.map(d => d.label));

/* find max */
$: maxBar = d3.greatest(data, d => d.value);

let xAxis;
let yAxis;

/* axes */
$: if (xAxis && yAxis) {
    d3.select(xAxis).call(d3.axisBottom(xScale));

    d3.select(yAxis).call(
        d3.axisLeft(yScale)
            .ticks(maxVal)
            .tickFormat(d => Number.isInteger(d) ? d : "")
    );
}
</script>