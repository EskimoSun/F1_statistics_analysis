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
    let svg;
    let lapsSvg;
    let selectedYear = 2022; //default
    let selectedRound = 1;
    let yearLegend;
    // Data filter
    let dataFilter;
    const driverColorStore = writable({});
    const commonColorScale = d3.scaleOrdinal().range(schemeSet3);

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
        /*
        const colorMap = $driverColorStore[year][round];
        return colorMap ? colorMap[driverId] : null;
        */
    }

    // Function to handle mouseover event on lines for the main chart
    function handleLineMouseOver(event, d) {
        const selectedDriverId = d[0];
        svg.selectAll(".year-line").style("opacity", (line) =>
            line[0] === selectedDriverId ? 1 : 0.1,
        );
        lapsSvg
            .selectAll(".round-line")
            .style("opacity", (line) =>
                line.driverId === selectedDriverId ? 1 : 0.1,
            );
        // Highlight legend text for both year and round
        // highlightLegendText(selectedDriverId, svg, yearLegend);
    }
    // Function to handle mouseout event on lines for the main chart
    function handleLineMouseOut() {
        lapsSvg.selectAll(".round-line").style("opacity", 1);
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
    }

    // Function to handle mouseover event on legend text for years
    function handleLegendMouseOver(event, d) {
        const selectedDriverId = d;
        // Select only the year lines
        lapsSvg
            .selectAll(".round-line")
            .style("opacity", (line) =>
                line.driverId === selectedDriverId ? 1 : 0.1,
            );
        highlightLegendText(selectedDriverId, lapsSvg, yearLegend);
    }
    // Function to handle mouseout event on legend text for years
    function handleLegendMouseOut() {
        lapsSvg.selectAll(".round-line").style("opacity", 1);
        resetLegendText(lapsSvg, yearLegend);
    }

    onMount(async () => {
        const margin = { top: 50, right: 30, bottom: 10, left: 60 },
            widthLapsChart = width - margin.left - margin.right,
            heightLapsChart = height * 0.75 - margin.top - margin.bottom;

        lapsSvg = select("#laps_dataviz")
            .append("svg")
            .attr("width", widthLapsChart)
            .attr("height", height)
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
            setDriverColors(selectedYear); // Set colors for drivers in the selected year
            updateChart(selectedYear, selectedRound);
        });
        // Listens to changes in selection
        lapsYearDropdown.on("change", function () {
            selectedYear = +this.value;
            updateRoundsDropdown(selectedYear);
            setDriverColors(selectedYear); // Set colors for drivers in the selected year
            updateChart(selectedYear, selectedRound);
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
        }

        // Initialize chart
        setDriverColors(selectedYear);
        updateChart(selectedYear, selectedRound);

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

            const lapsX = d3
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
            // x-axis label
            lapsSvg
                .append("text")
                .attr("class", "x-axis-label")
                .attr("x", widthLapsChart / 2)
                .attr("y",  0 - 25)
                .attr("text-anchor", "middle")
                .text("Lap Number");
            // Laptimes Per Race y-axis
            lapsSvg
                .append("g")
                .attr("class", "y-axis")
                .call(d3.axisLeft(lapsY));
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
                .attr("x1", (d, i) => (i % maxLines) * 50 + 10)
                .attr("y1", (d, i) => (i < maxLines ? -lineHeight + 45 : 45))
                .attr("x2", (d, i) => (i % maxLines) * 50 + 10)
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
                .attr("x", (d, i) => (i % maxLines) * 50 + 30)
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
                .style("stroke", "#FF004D")
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
                .on("mouseover", handleLapsLineMouseOver)
                .on("mouseout", () => handleLineMouseOut(yearLegend));
        }
    });
</script>

<!-- The container for the chart -->
<div id="chart-container">
    <div>
        <label for="laps_year_dropdown" style="color:#FAEF5D"
            >Select Year:</label
        >
        <select id="laps_year_dropdown"></select>
        <label for="round_dropdown" style="color:#FAEF5D">Select Round:</label>
        <select id="round_dropdown"></select>
        <div id="laps_dataviz"></div>
    </div>
</div>

<style>
    #chart-container {
        display: flex;
        justify-content: space-between;
    }

    #laps_dataviz {
        width: 100%;
    }
</style>
