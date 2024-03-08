<script>
  // Import necessary modules and functions
  import * as d3 from 'd3';
  import { onMount, afterUpdate } from 'svelte';
  import { schemeSet3 } from 'd3-scale-chromatic';
  import { select } from 'd3-selection';
  import { writable } from 'svelte/store';
  export let index, width, height;

  // Define all years
  const allYears = [2000, 2001, 2002, 2003, 2004, 2005, 2006, 2007, 2008, 2009,
    2010, 2011, 2012, 2013, 2014, 2015, 2016, 2017, 2018, 2019, 2020, 2021, 2022];
  // Initialize variables
  let data;
  let svg;
  let selectedYear = 2022; //default
  let selectedRound = 1;
  let yearLegend;
  // Data filter
  let dataFilter;
  const driverColorStore = writable({});
  const commonColorScale = d3.scaleOrdinal()
      .range(schemeSet3);

  // Function to set color for each driver in a specific year
  function setDriverColors(year) {
    const colorScale = d3.scaleOrdinal()
      .range(schemeSet3);
    
    const filteredData = data.filter(d => d.year == year);
    const uniqueConstructorIds = Array.from(new Set(filteredData.map(d => d.constructorId))); // Initialize uniqueConstructorIds

    const colorMap = {};

    uniqueConstructorIds.forEach((constructorId, index) => {
      const driversWithConstructor = filteredData.filter(d => d.constructorId === constructorId);

      driversWithConstructor.forEach(driver => {
        colorMap[driver.driverId] = colorScale(index);
      });
    });

    driverColorStore.set({ [year]: colorMap });
  }
  // Function to get color for a specific driver in a specific year
  function getDriverColor(year, driverId) {
    const colorMap = $driverColorStore[year];
    return colorMap ? colorMap[driverId] : null;
  }


  // Function to handle mouseover event on lines for the main chart
  function handleLineMouseOver(event, d) {
    const selectedDriverId = d[0];
    svg.selectAll(".year-line")
      .style("opacity", line => (line[0] === selectedDriverId ? 1 : 0.1));
    // Highlight legend text for both year and round
    highlightLegendText(selectedDriverId, svg, yearLegend);
  }
  // Function to handle mouseout event on lines for the main chart
  function handleLineMouseOut() {
    svg.selectAll(".year-line")
      .style("opacity", 1);
    // Reset legend text for both year and round
    resetLegendText(svg, yearLegend);
  }


  // Function to highlight legend text
  function highlightLegendText(driverId, chartSvg, legendGroup) {
    if (legendGroup) {  // Check if legendGroup is defined
      const selectedText = legendGroup.selectAll("text")
        .filter(d => d === driverId);
      legendGroup.selectAll("text")
        .style("opacity", d => (d === driverId ? 1 : 0.1));
      selectedText.raise(); // Bring the selected text to the front
    }
  }
  // Function to reset legend text
  function resetLegendText(chartSvg, legendGroup) {
    if (legendGroup) {
      legendGroup.selectAll("text")
        .style("opacity", 1);
    }
  }

  // Function to handle mouseover event on legend text for years
  function handleLegendMouseOver(event, d) {
    const selectedDriverId = d;
    // Select only the year lines
    svg.selectAll(".year-line")
      .style("opacity", line => (line[0] === selectedDriverId ? 1 : 0.1));
  }
  // Function to handle mouseout event on legend text for years
  function handleLegendMouseOut() {
    svg.selectAll(".year-line")
      .style("opacity", 1);
  }


  onMount(() => {
    const margin = { top: 10, right: 30, bottom: 30, left: 60 },
    widthPtsChart = width - margin.left - margin.right,
    heightPtsChart = height * 0.75 - margin.top - margin.bottom;

    svg = select("#year_dataviz")
      .append("svg")
      .attr("width", widthPtsChart)
      .attr("height", height)
      .append("g")
      .attr("transform", `translate(${margin.left},${margin.top})`);

    d3.csv("src/data/dataset.csv").then(function (loadedData) {
      data = loadedData;

      // Populate the year dropdown
      const yearDropdown = select("#year_dropdown");
        yearDropdown.selectAll('option')
          .data(allYears)
          .enter()
          .append('option')
          .text(d => d)
          .attr("value", d => d);
      yearDropdown.property("value", selectedYear);

      yearDropdown.on("change", function () {
        selectedYear = +this.value;
        setDriverColors(selectedYear); // Set colors for drivers in the selected year
        updateChart(selectedYear, selectedRound);
      });
      setDriverColors(selectedYear);
      updateChart(selectedYear);
    });

    function updateChart(selectedYear) {
      svg.selectAll("*").remove();
      dataFilter = data.filter(d => d.year == selectedYear);
      const sumstat = d3.group(dataFilter, d => d.driverId);

      const x = d3.scaleLinear()
        .domain(d3.extent(dataFilter, d => +d.round))
        .range([0, widthPtsChart * 0.94]);
      svg.append("g")
        .attr("class", "x-axis")
        .attr("transform", `translate(0, ${heightPtsChart})`)
        .call(d3.axisBottom(x).ticks(10));
      svg.append("text")
        .attr("class", "x-axis-label")
        .attr("x", widthPtsChart / 2)
        .attr("y", heightPtsChart + margin.bottom)
        .attr("text-anchor", "middle")
        .text("GP");

      const y = d3.scaleLinear()
        .domain([0, d3.max(dataFilter, d => +d.aggregrate_point)])
        .range([heightPtsChart, 0]);
      svg.append("g")
        .attr("class", "y-axis")
        .call(d3.axisLeft(y));
      svg.append("text")
        .attr("class", "y-axis-label")
        .attr("transform", "rotate(-90)")
        .attr("y", -margin.left)
        .attr("x", -heightPtsChart / 2)
        .attr("dy", "1em")
        .attr("text-anchor", "middle")
        .text("Pts");
      
      // Add legend for years
      yearLegend = svg.append("g")
        .attr("class", "year-legend")
        .attr("transform", `translate(${0},${heightPtsChart + 50})`);
      // Obtain corresponding color
      const color = commonColorScale
        .domain(dataFilter.map(d => d.driverId));
      const sortedDomain = color.domain().sort((a, b) => getDriverColor(selectedYear, a) > getDriverColor(selectedYear, b) ? 1 : -1);

      // Legend Lines
      const lineHeight = 50;
      const maxLines = Math.ceil(sortedDomain.length / 2);
      yearLegend.selectAll("line")
        .data(sortedDomain)
        .join("line")
        .attr("x1", (d, i) => i % maxLines * 50 + 10)
        .attr("y1", (d, i) => i < maxLines ? -lineHeight + 45 : 45)
        .attr("x2", (d, i) => i % maxLines * 50 + 10)
        .attr("y2", (d, i) => i < maxLines ? -lineHeight / 2 + 45 : lineHeight / 2 + 45)
        .style("stroke", d => getDriverColor(selectedYear, d))
        .style("stroke-width", 3)
        .style("cursor", "pointer")
        .on("mouseover", handleLegendMouseOver)
        .on("mouseout", handleLegendMouseOut);

      // Legend Texts
      yearLegend.selectAll("text")
        .data(sortedDomain)
        .join("text")
        .attr("x", (d, i) => i % maxLines * 50 + 30)
        .attr("y", (d, i) => i < maxLines ? -lineHeight + 55 : 55)
        .attr("dy", ".55em")
        .style("text-anchor", "middle")
        .text(d => data.find(entry => entry.driverId === d)?.code || '')
        .style("cursor", "pointer")
        .style("font-size", "8px")
        .on("mouseover", handleLegendMouseOver)
        .on("mouseout", handleLegendMouseOut);
      
      // Set style for all text and axis
      svg.selectAll('.y-axis line,.x-axis line').style('stroke', '#FF004D').style('stroke-width', 3);
      svg.selectAll(".y-axis text,.x-axis text").style('fill','#FAEF5D').style('font-size', '12px');
      svg.selectAll(".y-axis-label,.x-axis-label").style('fill','#FAEF5D').style('font-size', '18px');
      yearLegend.selectAll("text").style('fill','#FAEF5D').style('font-size', '15px');

      // Draw lines
      svg.selectAll(".line.year-line")
        .data(sumstat)
        .join("path")
        .attr("class", "line year-line") // Add class 'year-line'
        .attr("fill", "none")
        .attr("stroke", d => getDriverColor(selectedYear, d[0]))
        .attr("stroke-width", 2)
        .attr("d", d => d3.line()
          .x(d => x(d.round))
          .y(d => y(d.aggregrate_point))
        (d[1]))
        .style("opacity", 1)
        // Drawing animation
        .attr("stroke-dasharray", function() {
          const length = this.getTotalLength();
          return `${length} ${length}`;
        })
        .attr("stroke-dashoffset", function() {
          return this.getTotalLength();
        })
        .transition()
        .duration(1000)
        .attr("stroke-dashoffset", 0)
        .on("end", function() {
          // Apply event listeners after the transition
          d3.select(this)
          .on("mouseover", handleLineMouseOver)
          .on("mouseout", () => handleLineMouseOut(yearLegend)); // Pass legend as a parameter
        });
  
    }
  });

</script>

<!-- The container for the chart -->
<div id="chart-container">
  <div>
    <label for="year_dropdown" style="color:#FAEF5D">Select Year:</label>
    <select id="year_dropdown"></select>
    <div id="year_dataviz"></div>
  </div>
</div>

<style>
  #chart-container {
    display: flex;
    justify-content: space-between;
  }

  #year_dataviz {
    width: 100%;
  }

  
</style>