<script>
  import * as d3 from 'd3';
  import { onMount } from 'svelte';

  let data = [];
  let selectedGroup;

  onMount(async () => {
    const res = await fetch('data_full.csv');
    const csv = await res.text();

    // Parse the CSV with explicit data types
    data = d3.csvParse(csv, (d) => ({
      // Convert 'STANDARDINCOME' to a number
      ...d,
      STANDARDINCOME: +d.STANDARDINCOME,
    }));

    console.log(data);

    // Draw line plot for a specific occupation
    if (data.length > 0) {
      selectedGroup = data[0].Groups;
      drawLinePlot(data, selectedGroup);
    }
  });

  // Line Plot
  function drawLinePlot(data, selectedOccupation) {

    // Remove existing SVG elements
    d3.select("#my_dataviz").selectAll("*").remove();

    console.log("Selected Group:", selectedGroup);

    // Set the dimensions of the SVG
    const screenWidth = window.innerWidth;
    const screenHeight = window.innerHeight;

    // Adjust dimensions as needed
    const margin = { top: 60, right: 30, bottom: 90, left: 30 }; // Adjust left margin
    const padding = 60; // Increase the padding
    const width = screenWidth - margin.left - margin.right - (padding * 2);
    const height = screenHeight - margin.top - margin.bottom - (padding * 2);

    // Append the svg object to the body of the page
    var svg = d3.select("#my_dataviz")
      .append("svg")
      .attr("width", width + margin.left + margin.right + (padding * 2))
      .attr("height", height + margin.top + margin.bottom + (padding * 2))
      .append("g")
      .attr("transform", "translate(" + (margin.left + padding) + "," + (margin.top + padding) + ")");


    // Add title
    svg.append("text")
      .attr("x", width / 2)
      .attr("y", -90) // Adjusted y-coordinate
      .attr("dy", "0.75em")
      .style("text-anchor", "middle")
      .style("font-size", "1.5em") // Adjust font size as needed
      .text("Income Per Gender for " + selectedOccupation);

    // Filter data for the selected occupation and remove NaN values
    var filteredData = data.filter(d => d.Groups === selectedOccupation && !isNaN(d.STANDARDINCOME));

    // Group data by 'MULTYEAR' and 'SEX'
    var groupedData = d3.group(filteredData, d => d.MULTYEAR, d => d.SEX);

    // Calculate median values for male and female
    var medianValuesMale = Array.from(groupedData, ([year, sexValues]) => ({
      Year: year,
      Median: d3.median(sexValues.get('1') || [], d => d.STANDARDINCOME) || 0
    }));

    var medianValuesFemale = Array.from(groupedData, ([year, sexValues]) => ({
      Year: year,
      Median: d3.median(sexValues.get('2') || [], d => d.STANDARDINCOME) || 0
    }));

    // Set up X and Y scales
    var xScale = d3.scaleBand()
      .domain(medianValuesMale.map(d => d.Year))
      .range([0, width])
      .paddingInner(0.1);

    // Calculate the minimum value among male and female medians
    var minValue = d3.min([...medianValuesMale, ...medianValuesFemale], d => d.Median);
  

    // Set up Y scale based on the minimum value with a 1000 offset
    var yScale = d3.scaleLinear()
      .domain([Math.max(0, minValue - 3000), d3.max([...medianValuesMale, ...medianValuesFemale], d => d.Median)])
      .range([height, 0]);

    // Draw the lines for male and female
    svg.append("path")
      .datum(medianValuesMale)
      .attr("fill", "none")
      .attr("stroke", "#62aec5")
      .attr("stroke-width", 2)
      .attr("d", d3.line()
        .x(d => xScale(d.Year) + xScale.bandwidth() / 2)
        .y(d => yScale(d.Median))
      );

    svg.append("path")
      .datum(medianValuesFemale)
      .attr("fill", "none")
      .attr("stroke", "#e64072")
      .attr("stroke-width", 2)
      .attr("d", d3.line()
        .x(d => xScale(d.Year) + xScale.bandwidth() / 2)
        .y(d => yScale(d.Median))
      );

    // Draw circles for each data point on the male line
    svg.selectAll("circle.male")
      .data(medianValuesMale)
      .enter().append("circle")
      .attr("class", "male")
      .attr("cx", d => xScale(d.Year) + xScale.bandwidth() / 2)
      .attr("cy", d => yScale(d.Median))
      .attr("r", 5) // Adjust the radius as needed
      .attr("fill", "#62aec5") // Set the fill color for male circles
      .on("mouseover", function (event, d) {
      // Show tooltip on hover
      tooltip.transition()
        .duration(200)
        .style("opacity", .9);
      tooltip.html(`Year: ${d.Year}<br>Median (Male): ${d.Median}`)
        .style("left", (event.pageX + 10) + "px") // Adjust the left position
        .style("top", (event.pageY - 28) + "px"); // Adjust the top position
    })
    .on("mouseout", function (d) {
      // Hide tooltip on mouseout
      tooltip.transition()
        .duration(500)
        .style("opacity", 0);
    });

    // Draw circles for each data point on the female line
    svg.selectAll("circle.female")
      .data(medianValuesFemale)
      .enter().append("circle")
      .attr("class", "female")
      .attr("cx", d => xScale(d.Year) + xScale.bandwidth() / 2)
      .attr("cy", d => yScale(d.Median))
      .attr("r", 5) // Adjust the radius as needed
      .attr("fill", "#e64072") // Set the fill color for female circles
      .on("mouseover", function (event, d) {
      // Show tooltip on hover
      tooltip.transition()
        .duration(200)
        .style("opacity", .9);
      tooltip.html(`Year: ${d.Year}<br>Median (Female): ${d.Median}`)
        .style("left", (event.pageX + 10) + "px") // Adjust the left position
        .style("top", (event.pageY - 28) + "px"); // Adjust the top position
    })
    .on("mouseout", function (d) {
      // Hide tooltip on mouseout
      tooltip.transition()
        .duration(500)
        .style("opacity", 0);
    });

    // Draw X-axis
    svg.append("g")
      .attr("transform", "translate(0," + height + ")")
      .call(d3.axisBottom(xScale))
      .style("font-size", ".7em");

    // X-axis label
    svg.append("text")
      .attr("x", width / 2)
      .attr("y", height + margin.top - 30)
      .attr("dy", "0.75em")
      .style("text-anchor", "middle")
      .text("Year");

    // Draw Y-axis
    svg.append("g")
      .call(d3.axisLeft(yScale))
      .style("font-size", "0.7em");

    // Y-axis label
    svg.append("text")
      .attr("transform", "rotate(-90)")
      .attr("y", 0 - margin.left - 50)
      .attr("x", 0 - height / 2)
      .attr("dy", "1em")
      .style("text-anchor", "middle")
      .text("Total Pre-Tax Personal Income ($)");


    // Add a tooltip element
    var tooltip = d3.select("body")
      .append("div")
      .style("opacity", 0)
      .attr("class", "tooltip")
      .style("position", "absolute");
  }
</script>

<style>
  /* Add any styles if needed */
  #container {
    position: relative;
    min-height: 100vh;
    padding: 20px; /* Add padding for better spacing */
  }

  #dropdown {
    position: absolute;
    top: 0;
    left: 0;
  }

  #my_dataviz {
    width: 100vw;
    height: 100vh;
  }
</style>

<div id="container">
  <!-- Dropdown menu for selecting groups -->
  <select id="dropdown" bind:value={selectedGroup} on:change={() => drawLinePlot(data, selectedGroup)}>
    {#each [...new Set(data.map(d => d.Groups))] as group}
      <option value={group}>{group}</option>
    {/each}
  </select>

  <!-- Visualization container -->
  <div id="my_dataviz"></div>
</div>

<div style="position: absolute; top: 10px; right: 10px;">
  <svg width="100" height="70">
    <rect x="10" y="10" width="20" height="20" fill="#62aec5"></rect>
    <text x="40" y="15" dy="0.75em">Male</text>
    <rect x="10" y="40" width="20" height="20" fill="#e64072"></rect>
    <text x="40" y="45" dy="0.75em">Female</text>
  </svg>
</div>