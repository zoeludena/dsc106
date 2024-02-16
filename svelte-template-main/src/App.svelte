<script>
  import * as d3 from 'd3';
  import { onMount } from 'svelte';

  let data = [];

  onMount(async () => {
    const res = await fetch('data_full.csv');
    const csv = await res.text();

    // Parse the CSV with explicit data types
    data = d3.csvParse(csv, (d) => ({
      // Convert 'INCTOT' to a number
      ...d,
      INCTOT: +d.INCTOT,
    }));

    console.log(data);

    // Draw line plot for a specific occupation
    drawLinePlot(data, "Office and Administrative Support Occupations");
  });

  // Line Plot
  function drawLinePlot(data, selectedOccupation) {
    // Set the dimensions and margins of the graph
    var margin = { top: 10, right: 30, bottom: 30, left: 40 },
      width = 800 - margin.left - margin.right,
      height = 500 - margin.top - margin.bottom;

    // Append the svg object to the body of the page
    var svg = d3.select("#my_dataviz")
      .append("svg")
      .attr("width", width + margin.left + margin.right)
      .attr("height", height + margin.top + margin.bottom)
      .append("g")
      .attr("transform", "translate(" + margin.left + "," + margin.top + ")");

    // Filter data for the selected occupation and remove NaN values
    var filteredData = data.filter(d => d.Groups === selectedOccupation && !isNaN(d.INCTOT));

    // Group data by 'MULTYEAR' and 'SEX'
    var groupedData = d3.group(filteredData, d => d.MULTYEAR, d => d.SEX);

    // Calculate median values for male and female
    var medianValuesMale = Array.from(groupedData, ([year, sexValues]) => ({
      Year: year,
      Median: d3.median(sexValues.get('1') || [], d => d.INCTOT) || 0
    }));

    var medianValuesFemale = Array.from(groupedData, ([year, sexValues]) => ({
      Year: year,
      Median: d3.median(sexValues.get('2') || [], d => d.INCTOT) || 0
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
      .domain([Math.max(0, minValue - 1000), d3.max([...medianValuesMale, ...medianValuesFemale], d => d.Median)])
      .range([height, 0]);

    // Draw the lines for male and female
    svg.append("path")
      .datum(medianValuesMale)
      .attr("fill", "none")
      .attr("stroke", "blue")
      .attr("stroke-width", 2)
      .attr("d", d3.line()
        .x(d => xScale(d.Year) + xScale.bandwidth() / 2)
        .y(d => yScale(d.Median))
      );

    svg.append("path")
      .datum(medianValuesFemale)
      .attr("fill", "none")
      .attr("stroke", "pink")
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
      .attr("fill", "blue") // Set the fill color for male circles
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
      .attr("fill", "pink") // Set the fill color for female circles
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

    // Draw X-axis
    svg.append("g")
      .attr("transform", "translate(0," + height + ")")
      .call(d3.axisBottom(xScale));

    // Draw Y-axis
    svg.append("g")
      .call(d3.axisLeft(yScale));

    // Add a tooltip element
    var tooltip = d3.select("body")
      .append("div")
      .style("opacity", 0)
      .attr("class", "tooltip")
      .style("position", "absolute");

    // Add legend
    var legend = svg.append("g")
      .attr("transform", "translate(" + (width - 80) + ",10)");

    legend.append("rect")
      .attr("width", 20)
      .attr("height", 20)
      .attr("fill", "blue");

    legend.append("text")
      .attr("x", 30)
      .attr("y", 10)
      .attr("dy", "0.75em")
      .text("Male")
      .style("fill", "blue");

    legend.append("rect")
      .attr("y", 30) // Vertical offset
      .attr("width", 20)
      .attr("height", 20)
      .attr("fill", "pink");

    legend.append("text")
      .attr("x", 30)
      .attr("y", 40) // Vertical offset
      .attr("dy", "0.75em")
      .text("Female")
      .style("fill", "pink");
  }
</script>

<style>
  /* Add any styles if needed */
</style>

<div id="my_dataviz"></div>
