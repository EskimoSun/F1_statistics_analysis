<script>
    import * as d3 from 'd3';
    import { onMount } from 'svelte';

    export let index, width, height;
    const per_dom_stats = [
        { percentage_points: 'Points', ham: parseFloat((0.756410256 * 100).toFixed(2)), sch: parseFloat((0.822222222 * 100).toFixed(2)) },
        { percentage_wins: 'Wins', ham: parseFloat((0.523809524 * 100).toFixed(2)), sch: parseFloat((0.722222222 * 100).toFixed(2)) },
        { percentage_fl: 'Fastest Laps', ham: parseFloat((0.285714286 * 100).toFixed(2)), sch: parseFloat((0.555555556 * 100).toFixed(2)) },
        { percentage_pod: 'Podiums', ham: parseFloat((0.80952381 * 100).toFixed(2)), sch: parseFloat((0.833333333 * 100).toFixed(2)) },
        { percentage_poles: 'Poles', ham: parseFloat((0.238095238 * 100).toFixed(2)), sch: parseFloat((0.444444444 * 100).toFixed(2)) }
        ];

    const dom_stats = [
        { points: 'Points Count', ham: 413, sch: 148 },
        { wins: 'Wins Count', ham: 11, sch: 13 },
        { fl: 'Fastest Laps Count', ham: 6, sch: 10 },
        { pod: 'Podiums Count', ham: 17, sch: 15 },
        { poles: 'Poles Count', ham: 5, sch: 8 }
    ];

    let container;
    let svg;
    let isVisible;
    let tooltip;
    let allInitialized = false;
    let bar1;
    let bar2;
    let xScale;
    const barHeight = 50;

    $:if(index == 3){
        isVisible = true;
        if (allInitialized){
            toggleVisibility(isVisible);
        }
    }else{
        isVisible = false;
        if (allInitialized){
            toggleVisibility(isVisible);
        }
    }

    function toggleVisibility(makeVisible) {
        if (makeVisible) {
            // Animate to visible state
            bar1.transition()
                .duration(1000) // Duration of the transition
                .attr('width', d => Math.abs(xScale(d.ham) - xScale(0))) // Animate to final width
                .on('end', function() { // Once the transition is complete, attach event listeners
                    d3.select(this)
                    .on('mouseover', (event, d) => handleBarMouseOver(event, d, 'ham'))
                    .on('mouseout', (event, d) => handleBarMouseOut(event, d, 'ham'));
                });
            bar2.transition()
                .duration(1000) // Duration of the transition
                .attr('width', d => Math.abs(xScale(-d.sch) - xScale(0))) // Animate to final width
                .on('end', function() { // Once the transition is complete, attach event listeners
                    d3.select(this)
                    .on('mouseover', (event, d) => handleBarMouseOver(event, d, 'sch'))
                    .on('mouseout', (event, d) => handleBarMouseOut(event, d, 'sch'));
                });
        } else {
            // Animate back to width 0
            bar1.transition()
                .duration(1000) // Duration of the transition
                .attr('width', 0)
                .on('start', function() { // Before starting the hide transition, remove event listeners
                    d3.select(this).on('mouseover', null).on('mouseout', null);
                });
            bar2.transition()
                .duration(1000) // Duration of the transition
                .attr('width', 0)
                .on('start', function() { // Before starting the hide transition, remove event listeners
                    d3.select(this).on('mouseover', null).on('mouseout', null);
                });
        }
    }

    function handleBarMouseOver(event, d, key) {
        // Highlight the hovered bar
        const currentBar = d3.select(event.currentTarget);
        currentBar.attr('fill', 'orange');

        // Show a tooltip with information
        tooltip.transition()
            .duration(200)
            .style('opacity', 0.9);

        const barValue = Object.values(d)[0];

        tooltip.html(`${barValue}: ${d3.format(".2f")(d[key])}%`)
            .style('left', (event.pageX + 10) + 'px')
            .style('top', (event.pageY - 28) + 'px');

        // Add text inside the highlighted bar
        svg.append('text')
            .attr('class', 'highlighted-text')
            .attr('x', +currentBar.attr('x') + +currentBar.attr('width') / 2) // Convert to numbers
            .attr('y', +currentBar.attr('y') + barHeight / 2) // Convert to numbers
            .attr('dy', '.35em')
            .attr('text-anchor', 'middle')
            .attr('fill', 'black')  // Adjust the text color as needed
            .text(`${d3.format(".2f")(d[key])}%`);
    }

    function handleBarMouseOut(event, d, key) {
        // Reset the color of the bar to the original colors
        const currentBar = d3.select(event.currentTarget);
        const originalColor = key === 'ham' ? '#5CA4FF' : '#8CC84B';
        currentBar.attr('fill', originalColor);

        // Hide the tooltip
        tooltip.transition()
            .duration(500)
            .style('opacity', 0);

        // Remove the highlighted text when mouse out
        svg.selectAll('.highlighted-text').remove();
    }

    onMount(() => {
        const margin = { top: 20, right: 20, bottom: 20, left: 20 };
        const widthPlsChart = width * 0.95 - margin.left - margin.right;
        const heightPlsChart = height * 0.75 - margin.top - margin.bottom;
        const extraPadding = 50; // Adjust as needed
        // Select the container div after the component is mounted
        container = d3.select('#chart-container-3');

        // Remove any existing SVG elements inside the container
        container.selectAll('svg').remove();

        // Append a new SVG with the specified dimensions
        svg = container.append('svg')
            .attr('width', widthPlsChart + margin.left + margin.right)
            .attr('height', heightPlsChart + margin.top + margin.bottom)
            .append('g')
            .attr('transform', `translate(${margin.left},${margin.top})`);

        // Scales
        const maxDataValue = d3.max(per_dom_stats, d => Math.max(d.ham, d.sch));
        xScale = d3.scaleLinear()
            .domain([-maxDataValue - extraPadding, maxDataValue + extraPadding])
            .range([0, widthPlsChart]);

        const yScale = d3.scaleBand()
            .domain(per_dom_stats.map(d => Object.keys(d)[0]))
            .range([0, heightPlsChart])
            .padding(0.1);

        tooltip = d3.select("body").append("div")
            .attr("class", "tooltip")
            .style("opacity", 0);
        // Calculate the space between bar1 and bar2 for labelAttr
        const spaceBetweenBars = 70;

        // Draw bars
        bar1 = svg.selectAll('.bar1')
            .data(per_dom_stats)
            .enter().append('rect')
            .attr('class', 'bar1')
            .attr('x', d => xScale(Math.min(0, d.ham)) + spaceBetweenBars)
            .attr('y', d => yScale(Object.keys(d)[0]))
            .attr('width', 0)
            .attr('height', barHeight)
            .attr('fill', '#5CA4FF');

        // Add labels for bar1
        svg.selectAll('.label1')
            .data(per_dom_stats)
            .enter().append('text')
            .attr('class', 'label1')
            .attr('x', widthPlsChart - 30)
            .attr('y', (d, i) => yScale(Object.keys(d)[0]) + barHeight / 2)
            .attr('dy', '.35em')
            .attr('fill', "#FAEF5D")
            .attr('cursor', 'pointer')
            .attr('text-anchor', 'end')
            .each(function (d, i) {
                const label = d3.select(this);
                if (Object.values(per_dom_stats[i])[1] >= Object.values(per_dom_stats[i])[2]) {
                    label.append('tspan')
                        .html('<tspan>&#x1F3C6;</tspan>') // Unicode for the trophy symbol
                        .append('tspan')
                        .text(` ${Object.values(dom_stats[i])[0]}: ${Object.values(dom_stats[i])[1]}`);
                } else {
                    label.append('tspan')
                        .text(` ${Object.values(dom_stats[i])[0]}: ${Object.values(dom_stats[i])[1]}`);
                }
            });

        // Add labels between bar1 and bar2
        svg.selectAll('.labelAttr')
            .data(per_dom_stats)
            .enter().append('text')
            .attr('class', 'labelAttr')
            .attr('x', widthPlsChart/2) // Centered position
            .attr('y', d => yScale(Object.keys(d)[0]) + barHeight / 2)
            .attr('dy', '.35em')
            .attr('text-anchor', 'middle') // Center the text
            .attr('fill', "#FAEF5D")
            .text(d => Object.values(d)[0]);

        // Draw bars
        bar2 = svg.selectAll('.bar2')
            .data(per_dom_stats)
            .enter().append('rect')
            .attr('class', 'bar2')
            .attr('x', d => xScale(Math.min(0, -d.sch)) - spaceBetweenBars)
            .attr('y', d => yScale(Object.keys(d)[0]))
            .attr('width', 0)
            .attr('height', barHeight)
            .attr('fill', '#8CC84B');
        

        // Add labels for bar2
        svg.selectAll('.label2')
            .data(per_dom_stats)
            .enter().append('text')
            .attr('class', 'label2')
            .attr('x', margin.left + 30)
            .attr('y', (d, i) => yScale(Object.keys(d)[0]) + barHeight / 2)
            .attr('dy', '.35em')
            .attr('fill', "#FAEF5D")
            .attr('cursor', 'pointer')
            .each(function (d, i) {
                const label = d3.select(this);
                if (Object.values(per_dom_stats[i])[2] >= Object.values(per_dom_stats[i])[1]) {
                    label.append('tspan')
                        .html('<tspan>&#x1F3C6;</tspan>') // Unicode for the trophy symbol
                        .append('tspan')
                        .text(` ${Object.values(dom_stats[i])[0]}: ${Object.values(dom_stats[i])[2]}`);
                } else {
                    label.append('tspan')
                        .text(` ${Object.values(dom_stats[i])[0]}: ${Object.values(dom_stats[i])[2]}`);
                }
            });
    
        const scaleRight = d3.scaleLinear()
            .domain([maxDataValue + extraPadding, 0])
            .range([-spaceBetweenBars * 2, widthPlsChart/2 - spaceBetweenBars]);

        const xAxis1 = svg.append('g')
            .attr('class', 'x-axis')
            .attr('transform', 'translate(0,' + (barHeight + 500) + ')')
            .call(d3.axisTop(scaleRight).tickValues([20, 40, 60, 80])
            .tickFormat(d => (d <= 100) ? (d + '%') : null)); // Format ticks and hide those greater than 100

        // Customize the tick labels appearance
        xAxis1.selectAll('text')
            .style('text-anchor', 'end')
            .attr('dx', '-.8em')
            .attr('dy', '.15em');

        const scaleLeft = d3.scaleLinear()
            .domain([0, maxDataValue + extraPadding])
            .range([widthPlsChart / 2 + spaceBetweenBars, widthPlsChart + spaceBetweenBars]);

        const xAxis2 = svg.append('g')
            .attr('class', 'x-axis')
            .attr('transform', 'translate(0,' + (barHeight + 500) + ')')
            .call(d3.axisTop(scaleLeft).tickValues([20, 40, 60, 80])
            .tickFormat(d => (d <= 100) ? (d + '%') : null)); // Format ticks and hide those greater than 100

        // Customize the tick labels appearance
        xAxis2.selectAll('text')
            .style('text-anchor', 'end')
            .attr('dx', '-.8em')
            .attr('dy', '.15em');
        
        allInitialized = true;
    });
</script>

<div id="chart-container-3" class:visible={isVisible}></div>

<style>
    #chart-container-3{
        opacity: 0;
        visibility: hidden;
        transition: opacity 2s, visibility 2s;
    }
    #chart-container-3.visible{
        opacity: 1;
        visibility: visible;
    }
</style>