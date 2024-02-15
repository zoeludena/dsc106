<script>
  import * as d3 from 'd3';
  import { onMount } from 'svelte';

  let data = [];

  onMount(async () => {
    const res = await fetch('data_full.csv');
    const csv = await res.text();
    data = d3.csvParse(csv, d3.autoType);

    console.log(data);

    // Your D3 code here (fetching CSV, processing data, and drawing box plots)
    // ...

    // For example:
    drawBoxPlot(data);
  });

  function drawBoxPlot(data) {
    // Set the dimensions and margins of the graph
    var margin = { top: 10, right: 30, bottom: 30, left: 40 },
      width = 460 - margin.left - margin.right,
      height = 400 - margin.top - margin.bottom;

    // Append the svg object to the body of the page
    var svg = d3.select("#my_dataviz")
      .append("svg")
      .attr("width", width + margin.left + margin.right)
      .attr("height", height + margin.top + margin.bottom)
      .append("g")
      .attr("transform", "translate(" + margin.left + "," + margin.top + ")");


    // Group data by 'Groups' and 'SEX'
    var sumstat = d3.group(data, d => d.Groups, d => d.SEX);

    // Calculate quartiles, median, etc. based on the 'INCTOT' column
    var incomeColumn = 'INCTOT';
    sumstat.forEach((value, key1, map1) => {
      value.forEach((group, key2, map2) => {
        // Declare variables outside the loop
        let q1, median, q3, interQuantileRange, min, max;

        q1 = d3.quantile(group.map(function (g) { return +g[incomeColumn]; }).sort(d3.ascending), 0.25);
        median = d3.quantile(group.map(function (g) { return +g[incomeColumn]; }).sort(d3.ascending), 0.5);
        q3 = d3.quantile(group.map(function (g) { return +g[incomeColumn]; }).sort(d3.ascending), 0.75);
        interQuantileRange = q3 - q1;
        min = q1 - 1.5 * interQuantileRange;
        max = q3 + 1.5 * interQuantileRange;

        map2.set(key2, { q1: q1, median: median, q3: q3, interQuantileRange: interQuantileRange, min: min, max: max });
      });
      map1.set(key1, value);
    });

    // X-axis
    var x = d3.scaleBand()
      .domain(Array.from(sumstat.keys()))
      .range([0, width])
      .paddingInner(1)
      .paddingOuter(.5);

      svg.append("g")
      .attr("transform", "translate(0," + height + ")")
      .call(d3.axisBottom(x))
      .selectAll("text")
      .style("text-anchor", "end")
      .attr("dx", "-.8em")
      .attr("dy", ".15em")
      .attr("transform", "rotate(-45)");


    // Set the box width
    var boxWidth = 100;

    // Y-axis
    var y = d3.scaleLinear()
      .domain([0, d3.max(data, function (d) { return +d.INCTOT; })])
      .range([height, 0]);
    svg.append("g")
      .call(d3.axisLeft(y));

    // rectangle for the main box
    svg.selectAll("boxes")
      .data(Array.from(sumstat.entries()))
      .enter()
      .append("rect")
      .attr("x", function (d) { return x(d[0]) - boxWidth / 2; })
      .attr("y", function (d) { return y(d[1].q3); })
      .attr("height", function (d) { return y(d[1].q1) - y(d[1].q3); })
      .attr("width", boxWidth)
      .attr("stroke", "black")
      .style("fill", "#69b3a2");


    // Draw the box plots
    svg.selectAll("box")
      .data(sumstat)
      .enter()
      .append("g")
      .attr("transform", function (d) { return "translate(" + x(d.key) + " ,0)"; })
      .selectAll("rect")
      .data(function (d) { return d.values; })
      .enter()
      .append("rect")
      .attr("x", -boxWidth / 2)
      .attr("y", function (d) { return y(d.value.q3); })
      .attr("height", function (d) { return y(d.value.q1) - y(d.value.q3); })
      .attr("width", boxWidth)
      .attr("stroke", "black")
      .style("fill", function (d) { return color(d.key); });

    // ... add lines for median, whiskers, etc.
    // You can customize this part based on your box plot design
  }
</script>

<style>
  /* Add any styles if needed */
</style>

<div id="my_dataviz"></div>
