<script>
    import { onMount } from 'svelte';
  
    let data = [];
  
    onMount(async () => {
      const res = await fetch('/static/data.csv');
      const csv = await res.text();
  
      data = d3.csvParse(csv, d3.autoType);
  
      // Simplified D3.js code using the 'data' array
      const margin = { top: 10, right: 30, bottom: 30, left: 40 };
      const width = 460 - margin.left - margin.right;
      const height = 400 - margin.top - margin.bottom;
  
      const svg = d3.select("#my_dataviz")
        .append("svg")
          .attr("width", width + margin.left + margin.right)
          .attr("height", height + margin.top + margin.bottom)
        .append("g")
          .attr("transform", `translate(${margin.left},${margin.top})`);
  
      // Show the X scale
      const x = d3.scaleBand()
        .range([0, width])
        .domain(data.map(d => d.Species))
        .paddingInner(1)
        .paddingOuter(0.5);
  
      svg.append("g")
        .attr("transform", `translate(0,${height})`)
        .call(d3.axisBottom(x));
  
      // Show the Y scale
      const y = d3.scaleLinear()
        .domain([d3.min(data, d => d.Sepal_Length), d3.max(data, d => d.Sepal_Length)])
        .range([height, 0]);
  
      svg.append("g").call(d3.axisLeft(y));
  
      // Show individual points
      svg.selectAll("indPoints")
        .data(data)
        .enter()
        .append("circle")
          .attr("cx", d => x(d.Species))
          .attr("cy", d => y(d.Sepal_Length))
          .attr("r", 4)
          .style("fill", "white")
          .attr("stroke", "black");
    });
  </script>
  
  <style>
    /* Add any styles if needed */
  </style>
  
  <!-- HTML structure -->
  <div id="my_dataviz"></div>  