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
  let x;
  let y;
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

  function findClosestLine(snappedXValue, snappedYValue) {
    let closestLine;
    let closestDistance = Infinity;

    svg.selectAll(".line.year-line").each(function() {
        const path = d3.select(this);
        const lineData = path.datum()[1]; // Data points of the line
        // Iterate over each data point to find the closest y-coordinate for the given x-coordinate
        lineData.forEach(point => {
            if (point.round === snappedXValue) {
                const distance = Math.abs(point.aggregrate_point - snappedYValue);
                if (distance < closestDistance) {
                    closestDistance = distance;
                    closestLine = path;
                }
            }
        });
    });
    return closestLine;
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
    svg.selectAll(".line.year-line")
      .style("opacity", line => (line[0] === selectedDriverId ? 1 : 0.1));
    highlightLegendText(selectedDriverId, svg, yearLegend)
  }
  // Function to handle mouseout event on legend text for years
  function handleLegendMouseOut() {
    svg.selectAll(".line.year-line")
      .style("opacity", 1);
  }

  function updateTooltipRange(event) {
    const [tempx, tempy] = d3.pointer(event);
    const closestXValue = x.invert(tempx);
    const snappedXValue = Math.round(closestXValue);

    const closestYValue = y.invert(tempy);
    const snappedYValue = Math.round(closestYValue);

    if (snappedYValue >= 0 & snappedXValue >= 0) {
        const xRangeWidth = x(snappedXValue + 1) - x(snappedXValue);
        let tooltipRange = svg.select(".tooltip-range");

        if (tooltipRange.empty()) {
            tooltipRange = svg.append("rect")
                .attr("class", "tooltip-range")
                .attr("fill", "#ccc")
                .attr("opacity", 0.3);
        }

        tooltipRange
            .attr("x", x(snappedXValue) - xRangeWidth / 2)
            .attr("y", 0)
            .attr("width", xRangeWidth)
            .attr("height", height * 0.75 - 30);

        // Find the closest line
        const closestLine = findClosestLine(snappedXValue, snappedYValue);

        // Highlight the closest line if it's defined
        if (closestLine) {
            svg.selectAll(".line.year-line")
                .style("opacity", function() {
                    return (this === closestLine.node()) ? 1 : 0.1;
                });
            highlightLegendText(closestLine.datum()[0], svg, yearLegend);
        }

        // Add circles for all lines' y values under snappedXValue
        svg.selectAll(".circle-tooltip").remove(); // Remove existing circles
        svg.selectAll(".annotation-box").remove();
        svg.selectAll(".line.year-line").each(function() {
            const path = d3.select(this);
            const lineData = path.datum()[1]; // Data points of the line

            // Find the y value for snappedXValue
            const point = lineData.find(point => point.round === snappedXValue);
            const yValue = point?.aggregrate_point;
            const driverId = point?.driverId;

            const lastpoint = lineData.find(point => point.round + 1 === snappedXValue);
            const lastyValue = lastpoint?.aggregrate_point;

            const driverName = data.filter(
                (d) =>
                    d.year === selectedYear &&
                    d.driverId === driverId
                )
                .map((d) =>
                d.driverRef.replace(/_/g, ' ').replace(/\b\w/g, c => c.toUpperCase())
            )[0] || "N/A";

            const driverCode = data.filter(
                (d) =>
                    d.year === selectedYear &&
                    d.driverId === driverId
                )
                .map((d) =>
                d.code)[0] || "N/A";

            const round = data.filter(
                (d) =>
                    d.year === selectedYear &&
                    d.round === snappedXValue
                )
                .map((d) =>
                d.circuit_name)[0] || "N/A";

            if (yValue !== undefined && driverId !== undefined) {
                // Increase the radius if the circle corresponds to the closest line
                if (path.node() === closestLine.node()) {
                  // Add background rectangle for annotation
                  const textPadding = 5; // Padding around the text
                  const textHeight = 70; // Height of the text box
                  const textWidth = 180; // Width of the text box
                  const rectX = x(snappedXValue) - textWidth / 2; // X position of the rectangle
                  const rectY = y(yValue) - 10 - textHeight - textPadding; // Y position of the rectangle

                  svg.append("rect")
                    .attr("class", "annotation-box")
                    .attr("x", rectX > (width - 350) ? (width - 150) - textWidth : rectX < 0 ? 0 : rectX)
                    .attr("y", rectY > (height - 250) ? (height - 350) - textHeight : rectY < 0 ? 0 : rectY + 15)
                    .attr("width", textWidth)
                    .attr("height", textHeight)
                    .attr("fill", "rgba(240, 240, 240, 0.7)")
                    .attr("stroke-width", 1);

                  svg.append("text")
                      .attr("class", "annotation-box")
                      .attr("x", rectX > (width - 350) ? (width - 150) - textWidth / 2 : rectX < 0 ? textWidth / 2 : x(snappedXValue)) // Centering the text
                      .attr("y", rectY > (height - 250) ? (height - 350) - 4 * textHeight / 5 : rectY < 0 ? textHeight / 5 : y(yValue) - 55)
                      .text(`${driverName} (${driverCode})`)
                      .attr("text-anchor", "middle")
                      .attr("alignment-baseline", "middle")
                      .style("font-weight", "bold")
                      .style("font-size", "13px")
                      .style("fill", "black");

                    // Add annotation text for Number of Round
                    svg.append("text")
                        .attr("class", "annotation-box")
                        .attr("x", rectX > (width - 350) ? (width - 150) - textWidth/2 : rectX < 0 ? textWidth/2 : x(snappedXValue))
                        .attr("y", rectY > (height - 250) ? (height - 350) - (3 * textHeight / 4) + 15 : rectY < 0 ? (textHeight / 5) + 15 : y(yValue) - 40)
                        .text(`#${snappedXValue} ${round}`)
                        .attr("text-anchor", "middle")
                        .attr("alignment-baseline", "middle")
                        .style("font-size", "12px")
                        .style("fill", "black");

                    // Add annotation text for Cumulative Points
                    svg.append("text")
                        .attr("class", "annotation-box")
                        .attr("x", rectX > (width - 350) ? (width - 150) - textWidth/2 : rectX < 0 ? textWidth/2 : x(snappedXValue))
                        .attr("y", rectY > (height - 250) ? (height - 350) - 3 * textHeight / 4 + 30 : rectY < 0 ? textHeight / 5 + 30 : y(yValue) - 25)
                        .text(`Cumulative Points: ${yValue}`)
                        .attr("text-anchor", "middle")
                        .attr("alignment-baseline", "middle")
                        .style("font-size", "12px")
                        .style("fill", "black");

                    // Add annotation text for Current Points
                    svg.append("text")
                        .attr("class", "annotation-box")
                        .attr("x", rectX > (width - 350) ? (width - 150) - textWidth/2 : rectX < 0 ? textWidth/2 : x(snappedXValue))
                        .attr("y", rectY > (height - 250) ? (height - 350) - 3 * textHeight / 4 + 45 : rectY < 0 ? textHeight / 5 + 45 : y(yValue) - 10)
                        .text(`Current Round Points: ${lastyValue !== undefined ? yValue - lastyValue : yValue}`)
                        .attr("text-anchor", "middle")
                        .attr("alignment-baseline", "middle")
                        .style("font-size", "12px")
                        .style("fill", "black");


                    svg.append("circle")
                      .attr("class", "circle-tooltip")
                      .attr("cx", x(snappedXValue))
                      .attr("cy", y(yValue))
                      .attr("r", 6) // Default radius
                      .attr("fill", getDriverColor(selectedYear, driverId));
                } else {
                    svg.append("circle")
                        .attr("class", "circle-tooltip")
                        .attr("cx", x(snappedXValue))
                        .attr("cy", y(yValue))
                        .attr("r", 4) // Default radius
                        .attr("fill", getDriverColor(selectedYear, driverId))
                        .attr("opacity", 0.4);
                }
            }
        });
    } else {
        svg.select(".tooltip-range").remove();
        //svg.selectAll(".line.year-line").style("opacity", 1); // Restore opacity of all lines
        svg.selectAll(".annotation-box").remove(); // Remove existing annotation boxes
        svg.selectAll(".circle-tooltip").remove(); // Remove existing circles
    }
  }

  // Function to hide tooltip range when mouse leaves the chart area
  function hideTooltipRange() {
      svg.select(".tooltip-range").remove();
      svg.selectAll(".circle-tooltip").remove(); 
      resetLegendText(svg, yearLegend);
      svg.selectAll(".line.year-line").style("opacity", 1);
      svg.selectAll(".annotation-box").remove();
  }

  onMount(async () => {
    const margin = {top: 20, right: 30, bottom: 10, left: 60},
    widthPtsChart = width - margin.left - margin.right,
    heightPtsChart = height * 0.75 - margin.top - margin.bottom;

    svg = select("#year_dataviz")
      .append("svg")
      .attr("width", widthPtsChart)
      .attr("height", height)
      .append("g")
      .attr("transform", `translate(${margin.left},${margin.top})`);
    
      // Fetch and load data
    const race_info_csv = await fetch("dataset.csv");
    const race_info_text = await race_info_csv.text();
    data = d3.csvParse(race_info_text, d3.autoType);

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
        
    function updateChart(selectedYear) {
      svg.selectAll("*").remove();
      dataFilter = data.filter(d => d.year == selectedYear);
      const sumstat = d3.group(dataFilter, d => d.driverId);

      x = d3.scaleLinear()
        .domain(d3.extent(dataFilter, d => +d.round))
        .range([0, widthPtsChart * 0.94]);
      svg.append("g")
        .attr("class", "x-axis")
        .style("stroke-width", 3)
        .attr("transform", `translate(0, ${heightPtsChart})`)
        .call(d3.axisBottom(x).ticks(10));
      svg.append("text")
        .attr("class", "x-axis-label")
        .attr("x", widthPtsChart / 2)
        .attr("y", height * 0.75 + margin.bottom)
        .attr("text-anchor", "middle")
        .text("Round Number");

      y = d3.scaleLinear()
        .domain([0, d3.max(dataFilter, d => +d.aggregrate_point)])
        .range([heightPtsChart, 0]);
      svg.append("g")
        .attr("class", "y-axis")
        .style("stroke-width", 3)
        .call(d3.axisLeft(y));
      svg.append("text")
        .attr("class", "y-axis-label")
        .attr("transform", "rotate(-90)")
        .attr("y", -margin.left)
        .attr("x", -heightPtsChart / 2)
        .attr("dy", "1em")
        .attr("text-anchor", "middle")
        .text("Cumulative Points");
      
      // Add legend for years
      yearLegend = svg.append("g")
        .attr("class", "year-legend")
        .attr("transform", `translate(${0},${heightPtsChart + 70})`);
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
        .attr("x1", (d, i) => i % maxLines * 70 + 30)
        .attr("y1", (d, i) => i < maxLines ? -lineHeight + 45 : 45)
        .attr("x2", (d, i) => i % maxLines * 70 + 30)
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
        .attr("x", (d, i) => i % maxLines * 70 + 50)
        .attr("y", (d, i) => i < maxLines ? -lineHeight + 55 : 55)
        .attr("dy", ".55em")
        .style("text-anchor", "middle")
        .text(d => data.find(entry => entry.driverId === d)?.code || '')
        .style("cursor", "pointer")
        .style("font-size", "8px")
        .on("mouseover", handleLegendMouseOver)
        .on("mouseout", handleLegendMouseOut);
      
      
      svg.on("mousemove", updateTooltipRange)
        .on("mouseleave", hideTooltipRange);
      
      // Set style for all text and axis
      svg.selectAll('.y-axis line,.x-axis line').style('stroke', '#FAEF5D').style('stroke-width', 3);
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
        .attr("stroke-dashoffset", 0);
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
    flex-direction: column;
    justify-content: flex-start;
    align-items: center;
  }

  #year_dataviz {
    width: 100%; /* Ensure the content within year_dataviz spans the entire width */
  }
</style>