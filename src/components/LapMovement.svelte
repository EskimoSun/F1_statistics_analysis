<script>
    // Import necessary modules and functions
    import * as d3 from "d3";
    import { onMount, afterUpdate } from "svelte";
    import { schemeSet3 } from "d3-scale-chromatic";
    import { select } from "d3-selection";
    import { writable } from "svelte/store";
    export let index, width, height;

    // Define all years
    const allYears = [
        2000, 2001, 2002, 2003, 2004, 2005, 2006, 2007, 2008, 2009, 2010, 2011,
        2012, 2013, 2014, 2015, 2016, 2017, 2018, 2019, 2020, 2021, 2022,
    ];
    // Initialize variables
    let data;
    let lapsData;
    let lapsSvg;
    let lapsSvglap;
    let selectedYear = 2022; //default
    let selectedRound = 1;
    let yearLegend;
    let lapSliderValue = 1; // Default lap number
    let currentLap = 1;
    let maxLapNumber = 1;
    let xScaleLap;
    let lapsDataFilter;
    const driverColorStore = writable({});
    const commonColorScale = d3.scaleOrdinal().range(schemeSet3);
    let selectedDot = null;
    // Define the selected circle
    let selectedCircle = null;
    let lapsX;

    // Function to set color for each driver in a specific year
    function setDriverColors(year) {
        const colorScale = d3.scaleOrdinal().range(schemeSet3);

        const filteredData = data.filter((d) => d.year == year);
        const uniqueConstructorIds = Array.from(
            new Set(filteredData.map((d) => d.constructorId)),
        ); // Initialize uniqueConstructorIds

        const colorMap = {};

        uniqueConstructorIds.forEach((constructorId, index) => {
            const driversWithConstructor = filteredData.filter(
                (d) => d.constructorId === constructorId,
            );

            driversWithConstructor.forEach((driver) => {
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

    // Function to handle mouseout event on lines for the main chart
    function handleLineMouseOut() {
        lapsSvg.selectAll(".round-line").style("opacity", 1);
        lapsSvg.selectAll(".driver-code-annotation").remove();
        lapsSvg.selectAll(".annotation-background").remove();
        // Reset legend text for both year and round
        resetLegendText(lapsSvg, yearLegend);
    }

    // Function to highlight legend text
    function highlightLegendText(driverId, chartSvg, legendGroup) {
        if (legendGroup) {
            // Check if legendGroup is defined
            const selectedText = legendGroup
                .selectAll("text")
                .filter((d) => d === driverId);
            legendGroup
                .selectAll("text")
                .style("opacity", (d) => (d === driverId ? 1 : 0.1));
            selectedText.raise(); // Bring the selected text to the front
        }
    }
    // Function to reset legend text
    function resetLegendText(chartSvg, legendGroup) {
        if (legendGroup) {
            legendGroup.selectAll("text").style("opacity", 1);
        }
    }

    // Function to handle mouseover event on lines for rounds chart
    function handleLapsLineMouseOver(event, d) {
        const selectedDriverId = d.driverId;
        lapsSvg
            .selectAll(".line.round-line")
            .style("opacity", (line) =>
                line.driverId === selectedDriverId ? 1 : 0.1,
            );
        // Highlight legend text for both year and round
        highlightLegendText(selectedDriverId, lapsSvg, yearLegend);

                // Find the driver code
                const drivername = data
            .filter(
                (d) =>
                    d.year === selectedYear &&
                    d.round === selectedRound &&
                    d.driverId.toString() === selectedDriverId
            )
            .map((d) =>
                d.driverRef.replace(/_/g, ' ').replace(/\b\w/g, c => c.toUpperCase())
            )[0] || "N/A";
        
        // Find the driver code
        const drivercode = data
            .filter(
                (d) =>
                    d.year === selectedYear &&
                    d.round === selectedRound &&
                    d.driverId.toString() === selectedDriverId
            )
            .map((d) =>
                d.code)

        // Find the driver code
        const driverconst = data
            .filter(
                (d) =>
                    d.year === selectedYear &&
                    d.round === selectedRound &&
                    d.driverId.toString() === selectedDriverId
            )
            .map((d) =>
                d.name)

        // Calculate x and y coordinates for the left bottom of the graph
        const margin = 50; // Adjust the margin as needed
        const x = margin;
        const y = height * 0.6 - 100;

        // Add annotation at the left bottom of the graph
        lapsSvg.selectAll(".driver-code-annotation").remove(); // Remove existing annotations
        lapsSvg.selectAll(".annotation-background").remove();

        // Define dimensions for the background rectangle
        const textWidth = 220; // Adjust as needed
        const textHeight = 20; // Adjust as needed
        const rectPadding = 15; // Padding around the text

        // Add the background rectangle
        lapsSvg
        .append("rect")
        .attr("class", "annotation-background")
        .attr("x", x - rectPadding)
        .attr("y", y - rectPadding)
        .attr("width", textWidth + 2 * rectPadding)
        .attr("height", textHeight + 2 * rectPadding)
        .attr("fill", "rgba(240, 240, 240, 0.7)");

        // Add the text
        lapsSvg
            .append("text")
            .attr("class", "driver-code-annotation")
            .attr("x", x)
            .attr("y", y)
            .style("font-size", "15px")
            .selectAll("tspan")
            .data([
                { text: 'Driver: ' + drivername + ' (' + drivercode + ')', style: 'font-weight:bold' }, // Modify this line
                'Constructor: ' + driverconst
            ])
            .enter()
            .append("tspan")
            .attr("x", x)
            .attr("dy", (d, i) => i === 0 ? 0 : 20) // Adjust spacing between lines as needed
            .text((d) => d.text ? d.text : d) // Modify this line
            .attr("style", (d) => d.style ? d.style : null) // Modify this line
            .style("fill", "black");
    }

    function handleLegendMouseOver(event, d) {
        const selectedDriverId = d;

        // Select only the year lines
        lapsSvg
            .selectAll(".round-line")
            .style("opacity", (line) =>
                line.driverId === selectedDriverId ? 1 : 0.1,
            );
        highlightLegendText(selectedDriverId, lapsSvg, yearLegend);

        // Find the driver code
        const drivername = data
            .filter(
                (d) =>
                    d.year === selectedYear &&
                    d.round === selectedRound &&
                    d.driverId.toString() === selectedDriverId
            )
            .map((d) =>
                d.driverRef.replace(/_/g, ' ').replace(/\b\w/g, c => c.toUpperCase())
            )[0] || "N/A";
        
        // Find the driver code
        const drivercode = data
            .filter(
                (d) =>
                    d.year === selectedYear &&
                    d.round === selectedRound &&
                    d.driverId.toString() === selectedDriverId
            )
            .map((d) =>
                d.code)

        // Find the driver code
        const driverconst = data
            .filter(
                (d) =>
                    d.year === selectedYear &&
                    d.round === selectedRound &&
                    d.driverId.toString() === selectedDriverId
            )
            .map((d) =>
                d.name)

        // Calculate x and y coordinates for the left bottom of the graph
        const margin = 50; // Adjust the margin as needed
        const x = margin;
        const y = height * 0.6 - 100;

        // Add annotation at the left bottom of the graph
        lapsSvg.selectAll(".driver-code-annotation").remove(); // Remove existing annotations
        lapsSvg.selectAll(".annotation-background").remove();

        // Define dimensions for the background rectangle
        const textWidth = 220; // Adjust as needed
        const textHeight = 20; // Adjust as needed
        const rectPadding = 15; // Padding around the text

        // Add the background rectangle
        lapsSvg
        .append("rect")
        .attr("class", "annotation-background")
        .attr("x", x - rectPadding)
        .attr("y", y - rectPadding)
        .attr("width", textWidth + 2 * rectPadding)
        .attr("height", textHeight + 2 * rectPadding)
        .attr("fill", "rgba(240, 240, 240, 0.7)");

        // Add the text
        lapsSvg
            .append("text")
            .attr("class", "driver-code-annotation")
            .attr("x", x)
            .attr("y", y)
            .style("font-size", "15px")
            .selectAll("tspan")
            .data([
                { text: 'Driver: ' + drivername + ' (' + drivercode + ')', style: 'font-weight:bold' }, // Modify this line
                'Constructor: ' + driverconst
            ])
            .enter()
            .append("tspan")
            .attr("x", x)
            .attr("dy", (d, i) => i === 0 ? 0 : 20) // Adjust spacing between lines as needed
            .text((d) => d.text ? d.text : d) // Modify this line
            .attr("style", (d) => d.style ? d.style : null) // Modify this line
            .style("fill", "black");
    }

    // Function to handle mouseout event on legend text for years
    function handleLegendMouseOut() {
        lapsSvg.selectAll(".round-line").style("opacity", 1);
        lapsSvg.selectAll(".driver-code-annotation").remove();
        lapsSvg.selectAll(".annotation-background").remove();
        resetLegendText(lapsSvg, yearLegend);
    }

    onMount(async () => {
        const margin = { top: 50, right: 30, bottom: 10, left: 60 },
            widthLapsChart = width - margin.left - margin.right,
            heightLapsChart = height * 0.6 - margin.top - margin.bottom;

        lapsSvg = select("#laps_dataviz")
            .append("svg")
            .attr("width", widthLapsChart)
            .attr("height", height)
            .append("g")
            .attr("transform", `translate(${margin.left},${margin.top})`);

        lapsSvglap = select("#laps_dataviz2")
            .append("svg")
            .attr("width", widthLapsChart + margin.left + margin.right)
            .attr("height", heightLapsChart + margin.top + margin.bottom)
            .append("g")
            .attr("transform", `translate(${margin.left},${margin.top})`);

        // Fetch and load data
        const race_info_csv = await fetch("dataset.csv");
        const laps_csv = await fetch("laps_data.json");
        const race_info_text = await race_info_csv.text();
        const laps_text = await laps_csv.text();
        data = d3.csvParse(race_info_text, d3.autoType);
        lapsData = JSON.parse(laps_text);

        // Populate the rounds dropdown based on the selected year
        const roundDropdown = select("#round_dropdown");
        const rounds = Object.keys(
            lapsData[selectedYear.toString()] || {},
        );
        roundDropdown
            .selectAll("option")
            .data(rounds)
            .enter()
            .append("option")
            .text((d) => d)
            .attr("value", (d) => d);
        roundDropdown.property("value", selectedRound);
        // Populate the year dropdown

        const lapsYearDropdown = select("#laps_year_dropdown");
        lapsYearDropdown
            .selectAll("option")
            .data(allYears)
            .enter()
            .append("option")
            .text((d) => d)
            .attr("value", (d) => d);
        lapsYearDropdown.property("value", selectedYear);

        // Listens to changes in selection
        roundDropdown.on("change", function () {
            selectedRound = +this.value;
            selectedDot = null;
            currentLap = 1;
            lapSliderValue = 1;
            lapsSvglap.selectAll("*").remove();
            setDriverColors(selectedYear); // Set colors for drivers in the selected year
            updateChart(selectedYear, selectedRound);// Set colors for drivers in the selected year
            updateMaxLapNumber(selectedYear, selectedRound);
            updateLap(selectedYear, selectedRound, lapSliderValue);
            resetVerticalLine();
        });
        // Listens to changes in selection
        lapsYearDropdown.on("change", function () {
            selectedYear = +this.value;
            selectedDot = null;
            currentLap = 1;
            lapSliderValue = 1;
            lapsSvglap.selectAll("*").remove();
            updateRoundsDropdown(selectedYear);
            setDriverColors(selectedYear); // Set colors for drivers in the selected year
            updateChart(selectedYear, selectedRound);
            updateLap(selectedYear, selectedRound, lapSliderValue);
            updateMaxLapNumber(selectedYear, selectedRound);
        });

        function updateRoundsDropdown(year) {
            roundDropdown.selectAll("option").remove();
            const rounds = Object.keys(lapsData[year.toString()] || {});
            roundDropdown
                .selectAll("option")
                .data(rounds)
                .enter()
                .append("option")
                .text((d) => d)
                .attr("value", (d) => d);
            updateLap(selectedYear, selectedRound, lapSliderValue);
        }

        // Initialize chart
        setDriverColors(selectedYear);
        updateChart(selectedYear, selectedRound);
        updateLap(selectedYear, selectedRound, lapSliderValue);
        updateMaxLapNumber(selectedYear, selectedRound);

        function updateChart(selectedYear, selectedRound) {
            lapsSvg.selectAll("*").remove();
            let lapsDataFilter =
                lapsData[selectedYear.toString()][selectedRound];
            const lapData = Object.entries(lapsDataFilter).map(
                ([driverId, laps]) => ({
                    driverId,
                    laps: laps.map((lap) => ({
                        lap_num: lap.lap_num,
                        lap_time: lap.lap_time,
                    })),
                }),
            );

            lapsX = d3
                .scaleLinear()
                .domain(
                    d3.extent(
                        lapData.flatMap((d) => d.laps),
                        (lap) => +lap.lap_num,
                    ),
                )
                .range([0, widthLapsChart * 0.94]);

            const lapsY = d3
                .scaleLinear()
                .domain([
                    0,
                    d3.max(
                        lapData.flatMap((d) => d.laps),
                        (lap) => +lap.lap_time,
                    ),
                ])
                .range([0, heightLapsChart]);

            // Laptimes Per Race x-axis
            lapsSvg
                .append("g")
                .attr("class", "x-axis")
                .attr("transform", `translate(0, ${0})`)
                .call(d3.axisTop(lapsX).ticks(10));

            let LapName = data.filter(d => d.year === selectedYear && d.round === selectedRound).map((d) => d.competition_name);
            // x-axis label
            lapsSvg
                .append("text")
                .attr("class", "x-axis-label")
                .attr("x", (widthLapsChart - 100) / 2)
                .attr("y",  0 - 25)
                .attr("text-anchor", "middle")
                .text(LapName[0] + "'s Lap Number");
            // Laptimes Per Race y-axis
            const yAxisFormat = d3.format(".1f");

            lapsSvg
                .append("g")
                .attr("class", "y-axis")
                .attr("transform", `translate(0, ${0})`)
                .call(d3.axisLeft(lapsY).tickFormat(d => `+${yAxisFormat(d)}`));

            // y-axis label
            lapsSvg
                .append("text")
                .attr("class", "y-axis-label")
                .attr("transform", "rotate(-90)")
                .attr("y", -margin.left)
                .attr("x", -heightLapsChart / 2)
                .attr("dy", "1em")
                .attr("text-anchor", "middle")
                .text("Laptime Difference");
            // Add legend for years
            yearLegend = lapsSvg
                .append("g")
                .attr("class", "year-legend")
                .attr("transform", `translate(${0 + 20},${heightLapsChart + 10})`);
            // Obtain corresponding color
            const color = commonColorScale.domain(
                Object.keys(
                    lapsData[selectedYear.toString()][
                        selectedRound.toString()
                    ] || {},
                ),
            );
            const sortedDomain = color
                .domain()
                .sort((a, b) => (getDriverColor(selectedYear, a.toString()) > getDriverColor(selectedYear, b.toString()) ? 1 : -1));
            // Legend Lines
            const lineHeight = 50;
            const maxLines = Math.ceil(sortedDomain.length / 2);

            yearLegend
                .selectAll("line")
                .data(sortedDomain)
                .join("line")
                .attr("x1", (d, i) => (i % maxLines) * 70 + 10)
                .attr("y1", (d, i) => (i < maxLines ? -lineHeight + 45 : 45))
                .attr("x2", (d, i) => (i % maxLines) * 70 + 10)
                .attr("y2", (d, i) =>
                    i < maxLines ? -lineHeight / 2 + 45 : lineHeight / 2 + 45,
                )
                .style("stroke", (d) => getDriverColor(selectedYear, d.toString()))
                .style("stroke-width", 3)
                .style("cursor", "pointer")
                .on("mouseover", handleLegendMouseOver)
                .on("mouseout", handleLegendMouseOut);
            
            // Legend Texts
            yearLegend
                .selectAll("text")
                .data(sortedDomain)
                .join("text")
                .attr("x", (d, i) => (i % maxLines) * 70 + 30)
                .attr("y", (d, i) => (i < maxLines ? -lineHeight + 55 : 55))
                .attr("dy", ".55em")
                .style("text-anchor", "middle")
                .text(
                    (d) =>
                        data.find((entry) => entry.driverId == d)?.code || "",
                )
                .style("cursor", "pointer")
                .style("font-size", "8px")
                .on("mouseover", handleLegendMouseOver)
                .on("mouseout", handleLegendMouseOut);
            // Set style for text and axis
            lapsSvg
                .selectAll(
                    ".y-axis path,.y-axis line,.x-axis path, .x-axis line",
                )
                .style("stroke", "#FAEF5D")
                .style("stroke-width", 3);
            yearLegend
                .selectAll("text")
                .style("fill", "#FAEF5D")
                .style("font-size", "15px");
            lapsSvg
                .selectAll(".y-axis text,.x-axis text")
                .style("fill", "#FAEF5D")
                .style("font-size", "12px");
            lapsSvg
                .selectAll(".y-axis-label,.x-axis-label")
                .style("fill", "#FAEF5D")
                .style("font-size", "18px");

            // Draw lines for the laps chart
            lapsSvg
                .selectAll(".line.round-line")
                .data(lapData)
                .join("path")
                .attr("class", "line round-line")
                .attr("fill", "none")
                .attr("stroke", (d) =>
                    getDriverColor(selectedYear, d.driverId),
                ) // Use getDriverColor
                .attr("stroke-width", 2)
                .attr("d", (d) => {
                    if (d.laps.length > 0) {
                        return d3
                            .line()
                            .x((lap) => lapsX(lap.lap_num))
                            .y((lap) => lapsY(lap.lap_time))(d.laps);
                    } else {
                        return null;
                    }
                })
                // Drawing animation
                .attr("stroke-dasharray", function() {
                const length = this.getTotalLength();
                return `${length} ${length}`;
                })
                .attr("stroke-dashoffset", function() {
                return this.getTotalLength();
                })
                .transition()
                .duration(500)
                .attr("stroke-dashoffset", 0)
                .on("end", function() {
                // Apply event listeners after the transition
                d3.select(this)
                .on("mouseover", handleLapsLineMouseOver)
                .on("mouseout", () => handleLineMouseOut(yearLegend));
                });

                lapsSvg.append('text')
                    .attr("class", "writing")
                    .attr("x", 0)
                    .attr("y", height * 0.75)
                    .attr("font-size", "20px")
                    .style("fill", "#FAEF5D")
                    .selectAll("tspan")
                    .data(["click on a circle", " to follow the driver's path"]) // Split the text into two parts
                    .enter()
                    .append("tspan")
                    .text((d, i) => d)
                    .attr("font-weight", (d, i) => i === 0 ? "bold" : "normal"); // Bold for the first part, normal for the second
            }
    });
    // Function to update chart based on slider value
    function updateChartWithSlider() {
        updateLap(selectedYear, selectedRound, lapSliderValue);
        updateVerticalLine(lapSliderValue);
        if (lapSliderValue == maxLapNumber) {
            resetVerticalLine();
        }
    }

    function updateAnnotation(lapNum, lapTime, chartWidth, driverId) {
        // Remove any existing annotation group
        lapsSvg.selectAll(".annotation-group").remove();

        // Define the offset for the annotation
        const annotationOffset = 10;

        // Create a group for the annotation
        const annotationGroup = lapsSvg.append("g")
            .attr("class", "annotation-group")
            .attr("transform", `translate(${chartWidth + annotationOffset}, 1)`); // Adjusted x position with offset

        // Background rectangle for the annotation
        annotationGroup.append("rect")
            .attr("class", "annotation-bg")
            .attr("x", 20)
            .attr("y", height * 0.65 - 30)
            .attr("width", chartWidth / 3 - annotationOffset) // Adjust the width as needed
            .attr("height", 75) // Adjust the height as needed
            .attr("fill", "rgba(255, 255, 255, 0.7)"); // Semi-transparent white background

        // Find driver code for the current lap
        const driver = data.find(d => d.year === selectedYear && d.round === selectedRound && d.driverId.toString() === driverId);
        const driverCode = driver ? driver.code : "N/A"; // If driver not found, display "N/A"
        const driverName = driver ? driver.driverRef.replace(/_/g, ' ').replace(/\b\w/g, c => c.toUpperCase()) : "N/A"; // If driver not found, display "N/A"

        // Append driver code as text with formatting
        annotationGroup.append("text")
            .attr("class", "annotation-driver")
            .attr("x", 30) // Adjust the x position as needed
            .attr("y", height * 0.65 - 10) // Adjust the y position as needed
            .attr("font-weight", "bold") // Bold font
            .attr("font-size", "16px") // Larger font size
            .text(driverName + " (" + driverCode + ")"); // Driver code

        // Append lap number as text
        annotationGroup.append("text")
            .attr("class", "annotation-lap")
            .attr("x", 30) // Adjust the x position as needed
            .attr("y", height * 0.65 + 10) // Adjust the y position as needed
            .text("Current Lap: " + lapNum); // Lap number

        // Append lap time as text
        annotationGroup.append("text")
            .attr("class", "annotation-time")
            .attr("x", 30) // Adjust the x position as needed
            .attr("y", height * 0.65 + 30) // Adjust the y position as needed
            .text("Gap to Leader: " + (lapTime*60).toFixed(2) + 's'); // Lap time
    }

    // Function to update the position of the small circle based on the selected lap
    function updateDot(lapNum, lapTime, chartWidth, xScaleLap) {
        // Check if there is a selected dot
        if (selectedDot) {
            // Transition the selected dot to the new position
            selectedDot.transition()
                .duration(500) // Transition duration
                .attr("cx", xScaleLap(lapTime)); // Move to the correct x position
        } else {
            // If there is no selected dot, create a new one
            selectedDot = lapsSvglap.append("circle")
                .attr("class", "dot")
                .attr("cy", -20) // Adjust this value according to your layout
                .attr("r", 5) // Adjust the radius of the small circle
                .attr("fill", "#FAEF5D"); // Adjust the color of the small circle

            // Transition the new dot to the new position
            selectedDot.transition()
                .duration(500) // Transition duration
                .attr("cx", xScaleLap(lapTime)); // Move to the correct x position
        }
    }

    function resetVerticalLine() {
        lapsSvg.selectAll(".vertical-line, .left-rect, .right-rect").remove();
    }

    function updateVerticalLine(lapNum) {
        // Remove existing vertical line and rectangles
        lapsSvg.selectAll(".vertical-line, .left-rect, .right-rect").remove();

        // Append a new line as the vertical line
        lapsSvg.append("line")
            .attr("class", "vertical-line")
            .attr("x1", lapsX(lapNum))
            .attr("y1", 0)
            .attr("x2", lapsX(lapNum))
            .attr("y2", height * 0.6 - 60)
            .attr("stroke", "#FAEF5D")
            .attr("stroke-width", 2);

        // Append left rectangle
        lapsSvg.append("rect")
            .attr("class", "left-rect")
            .attr("x", 0)
            .attr("y", 0)
            .attr("width", Math.max(0, lapsX(lapNum) - 10)) // Ensures width is non-negative
            .attr("height", height * 0.6 - 60)
            .attr("fill", "rgba(32, 28, 68, 0.7)");

        // Append right rectangle
        lapsSvg.append("rect")
            .attr("class", "right-rect")
            .attr("x", lapsX(lapNum) + 10)
            .attr("y", 0)
            .attr("width", width - lapsX(lapNum) - 10)
            .attr("height", height * 0.6 - 60)
            .attr("fill", "rgba(32, 28, 68, 0.7)");
    }


    function updateLap(selectedYear, selectedRound, selectedLap) {
        // Calculate width for the chart and annotation
        const chartWidth = (width - 60) * 0.9;
        currentLap = selectedLap;

        // Remove all circles and texts before rendering new ones
        //lapsSvg.selectAll(".lap-circle").remove();
        //lapsSvg.selectAll(".driver-text").remove();

        lapsDataFilter = lapsData[selectedYear.toString()][selectedRound];

        const lapData = Object.entries(lapsDataFilter).map(
            ([driverId, laps]) => ({
                driverId,
                laps: laps.map((lap) => ({
                    lap_num: lap.lap_num,
                    lap_time: lap.lap_time,
                })),
            }),
        );

        // Filter laps based on selected lap number
        lapData.forEach(driver => {
            driver.laps = driver.laps.filter(lap => lap.lap_num === selectedLap);
        });

        // Find the largest lap time
        const maxLapTime = d3.max(lapData.flatMap(driver => driver.laps.map(lap => lap.lap_time)));


        // Define scale for x position of circles
        xScaleLap = d3.scaleLinear()
            .domain([0, maxLapTime])
            .range([chartWidth, 0]); // Adjusted to fit the new chart width

        // Update the position of the selected circle based on the lap progress
        updateSelectedCirclePosition(selectedLap);
 
        function updateSelectedCirclePosition(selectedLap) {
            if (selectedCircle) {
                const selectedCx = parseFloat(selectedCircle.attr("cx")); // Get the x-coordinate of the selected circle
                const circles = lapsSvglap.selectAll(".lap-circle");
                circles.each(function() {
                    const circle = d3.select(this);
                    const lapNumber = circle.data()[0].lap_num;
                    if (lapNumber !== selectedLap) {
                        const initialCx = parseFloat(circle.attr("data-initial-cx")); // Get the initial x-coordinate of the circle
                        circle.attr("cx", initialCx); // Restore the initial x-coordinate of the circle
                    }
                });
            }
        }
    
        // Function to handle circle click event
        function handleCircleClick(event, d, chartWidth, driverId) {
            // Update the annotation
            updateAnnotation(d.lap_num, d.lap_time, chartWidth * 0.75, driverId);
            updateDot(d.lap_num, d.lap_time, chartWidth, xScaleLap); // Pass lap_num and xScale as parameters

            // Store the selected circle for reference
            selectedCircle = d3.select(event.currentTarget);
        }


        // Render circles and text for each driver's lap time
        lapData.forEach(driver => {
            // Find the corresponding driverRef for all circles
            const driverIdString = driver.driverId.toString();

            const drivercode = data.filter(d => d.year === selectedYear && d.round === selectedRound && d.driverId.toString() === driverIdString)
                .map(d => d.code);

            const circles = lapsSvglap.selectAll(".lap-circle-" + driver.driverId)
                .data(driver.laps);

            // Exit old circles smoothly
            circles.exit()
                .transition()
                .duration(500)
                .attr("cx", 0) // Move them to the right side
                .remove();

            // Enter new circles
            let enteredCircles = circles.enter()
                .append("circle")
                .attr("class", "lap-circle lap-circle-" + driver.driverId)
                .attr("cx", chartWidth) // Start from the right side
                .attr("cy", 10) // Adjust this value according to your layout
                .attr("r", 18)
                .attr("fill", d => getDriverColor(selectedYear, driver.driverId))
                .attr("stroke", "white")
                .on("click", function(event, d) {
                    handleCircleClick(event, d, chartWidth, driver.driverId);
                }); // Call handleCircleClick on click

            // Update existing circles
            circles.merge(enteredCircles)
                .each(function(d) {
                    const circle = d3.select(this);
                    const initialCx = xScaleLap(d.lap_time); // Initial x-coordinate is the current x position
                    circle.attr("data-initial-cx", initialCx); // Store the initial x-coordinate as a data attribute
                    circle.transition()
                        .duration(500) // Transition duration
                        .attr("cx", initialCx) // Move to the correct x position
                        .attr("fill", getDriverColor(selectedYear, driver.driverId)); // Update fill color
            });

            // Update the position of the selected circle based on the lap progress
            updateSelectedCirclePosition(selectedLap);

            const texts = lapsSvglap.selectAll(".driver-text-" + driver.driverId)
                .data(driver.laps);

            // Exit old texts smoothly
            texts.exit()
                .transition()
                .duration(500)
                .attr("x", 0) // Move them to the right side
                .remove();

            // Enter new texts
            texts.enter()
                .append("text")
                .attr("class", "driver-text driver-text-" + driver.driverId)
                .attr("x", chartWidth) // Start from the right side
                .attr("y", 10) // Adjust this value according to your layout
                .text(drivercode[0]) // Assuming all driverRefs are the same for a driver
                .attr("text-anchor", "middle")
                .attr("alignment-baseline", "middle")
                .attr("fill", "rgba(170, 51, 120, 0.9)")
                .attr("font-size", "15px")
                .merge(texts) // Merge old and new texts
                .transition() // Apply transition
                .duration(500) // Transition duration
                .attr("x", d => xScaleLap(d.lap_time)); // Move to the correct x position
        });

        // Define custom tick format function
        const customTickFormat = (value) => {
            return value === 0 ? 'Leader' : '+' + value;
        };

        // Update the x-axis creation to use the custom tick format
        lapsSvglap.selectAll(".x-axis").remove(); // Remove previous axis
        lapsSvglap.append("g")
            .attr("class", "x-axis")
            .attr("transform", "translate(0, 50)") // Adjust vertical position as needed
            .attr("stroke-width", 3)
            .style("font-size", "12px")
            .call(d3.axisBottom(xScaleLap).tickFormat(customTickFormat)); // Create the axis based on the xScale with custom tick format

        // Update the position of the small circle based on the lap progress
        updateSelectedCirclePosition(selectedLap);

        // Initialize the annotation with the lap data of the selected driver
        if (selectedCircle) {
            // Extract driverId from the selected circle's class attribute
            const driverId = selectedCircle.attr("class").split(" ").find(className => className.startsWith("lap-circle-")).replace("lap-circle-", "");

            // Find the selected driver's data
            const selectedDriver = lapData.find(driver => driver.driverId === driverId);
            
            // Check if the selected driver exists and has laps recorded
            if (selectedDriver && selectedDriver.laps.length > 0) {
                // Get the first lap data of the selected driver
                const firstLap = selectedDriver.laps[0];

                // Update the annotation and dot with the first lap data
                updateAnnotation(firstLap.lap_num, firstLap.lap_time, chartWidth * 0.75, driverId);
                updateDot(firstLap.lap_num, firstLap.lap_time, chartWidth, xScaleLap);
                updateSelectedCirclePosition(selectedLap);
                updateVerticalLine(firstLap.lap_num);
            }
        }
    }

    // Function to update the maximum lap number based on selected year and round
    function updateMaxLapNumber(selectedYear, selectedRound) {
        const lapsDataFilter = lapsData[selectedYear.toString()][selectedRound];
        maxLapNumber = Math.max(...Object.values(lapsDataFilter).map(laps => laps.length));
    }

    let autoUpdateInterval;
    let isAutoUpdateRunning = false;

    function updateChartAutomatically() {
        const transitionDuration = 500; // milliseconds

        clearInterval(autoUpdateInterval); // Clear the interval to prevent multiple intervals running simultaneously

        autoUpdateInterval = setInterval(() => {
            if (currentLap <= maxLapNumber) {
                lapSliderValue = currentLap;
                updateLap(selectedYear, selectedRound, currentLap);
                updateVerticalLine(currentLap);
                currentLap++;
            } else {
                clearInterval(autoUpdateInterval); // Stop the interval when all laps are shown
                currentLap = 1; // Reset the lap number to 1
                isAutoUpdateRunning = false; // Reset the flag
                resetVerticalLine();
            }
        }, transitionDuration); // Smooth transition between each update

        isAutoUpdateRunning = true; // Set the flag to indicate the auto update is running
    }

    function stopAutomaticUpdate() {
        clearInterval(autoUpdateInterval); // Clear the interval to stop automatic updates
        isAutoUpdateRunning = false; // Reset the flag
    }
</script>

<div id="chart-container">
    <div class="dropdown-container">
        <label for="laps_year_dropdown" style="color:#FAEF5D">Select Year:</label>
        <select id="laps_year_dropdown"></select>
        &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
        <label for="round_dropdown" style="color:#FAEF5D">Select Round:</label>
        <select id="round_dropdown"></select>
    </div>
    <div id="laps_dataviz"></div>
    <div style="display: flex; align-items: center; justify-content: center; margin-top: 40px;">
        <span style="margin-left: 10px;">Current Lap: {lapSliderValue}</span>
        <input type="range" min="1" max={maxLapNumber} bind:value={lapSliderValue} on:input={updateChartWithSlider} style="margin-top: 10px;">
        <!-- Play icon for automatic update -->
        <!-- svelte-ignore a11y-click-events-have-key-events -->
        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" width="32" height="32" fill="currentColor" stroke="none" on:click={updateChartAutomatically}>
            <path d="M8 5v14l11-7z" />
        </svg>
        <!-- Stop icon for stopping automatic update -->
        <!-- svelte-ignore a11y-click-events-have-key-events -->
        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" width="32" height="32" fill="currentColor" stroke="none" on:click={stopAutomaticUpdate}>
            <path d="M8 5h3v14h-3zm5 0h3v14h-3z"/>
        </svg>
    </div>
    <div id="laps_dataviz2"></div>
</div>



<style>
    #chart-container {
    display: flex;
    flex-direction: column;
    justify-content: flex-start;
    align-items: center;
    height: 100%;
    }

    .dropdown-container {
        display: flex;
        justify-content: center;
        align-items: center;
    }

    #laps_dataviz,
    #laps_dataviz2 {
        width: 100%;
        height: 55%; /* 75% divided by 2 */
    }
</style>