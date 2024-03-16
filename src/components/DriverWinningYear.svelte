<script>
    import * as d3 from 'd3';
    import { onMount } from 'svelte';

    export let index, width, height;
    const per_win_stats = [
        { percentage_points: 'Points', ham: parseFloat((0.76169265 * 100).toFixed(2)), sch: parseFloat((0.686440678 * 100).toFixed(2)) },
        { percentage_wins: 'Wins', ham: parseFloat((0.503703704 * 100).toFixed(2)), sch: parseFloat((0.550847458 * 100).toFixed(2)) },
        { percentage_fl: 'Fastest Laps', ham: parseFloat((0.281481481 * 100).toFixed(2)), sch: parseFloat((0.36440678 * 100).toFixed(2)) },
        { percentage_pod: 'Podiums', ham: parseFloat((0.762962963 * 100).toFixed(2)), sch: parseFloat((0.745762712 * 100).toFixed(2)) },
        { percentage_poles: 'Poles', ham: parseFloat((0.459259259 * 100).toFixed(2)), sch: parseFloat((0.423728814 * 100).toFixed(2)) }
        ];

    const stats_annotation = {
        percentage_points: 'Other than the biased measures caused by the different points scoring systems, Schumacher has also had way more retired races than Hamilton, contributing to his lower points scored. Over each of their championship seasons, Michael Schumacher had accumulated 14 race DNFs (retired status), and 2 disqualifications, while Hamilton had only recieved a mere 6 DNFs over the span of 7 championship years. This demonstrates that Hamilton had a generally more reliable car than Schumacher, while Schumacher might have had a more aggressive driving style when compared to Hamilton.',
        percentage_wins: "When we focus only on the 7 championship seasons, Michael's win rates had exceeded that of Lewis'. Most of these wins come from 2000-2004 seasons, when he was the leading driver at Ferrari. This fact reflects how dominant he and his team were at the height of their greatness. ",
        percentage_fl: "Similar to the career stats comparison, Schumacher still has an edge over Hamilton in the percentage fastest laps attributed to his well-known race pace. Additionally, obtaining the fastest lap in a race at Schumacher's time was not rewarded with one championship point, so it was purely decided by the competitiveness of the race and the sheer pace he delivered.",
        percentage_pod: "Both drivers had an unbelievable amount of podium finishes, showing that both of them are incredibly consistent across all types of race tracks. ",
        percentage_poles: "As we focus down to 7 years, Schumacher's pole percentage has gone up 20% and Hamilton's has gone around 15%. The one-lap speed often is a showdown for the performance of the cars and how it stacks up against other team's cars. In their dominant eras, both drivers' cars were big contributing factors to their success. ",
    }

    const win_stats = [
        { points: '#Points', ham: 2394, sch: 810 },
        { wins: '#Wins', ham: 68, sch: 65 },
        { fl: '#Fastest Laps', ham: 38, sch: 43 },
        { pod: '#Podiums', ham: 103, sch: 88 },
        { poles: '#Poles', ham: 62, sch: 50 }
    ];

    const total_win_stats = [
        { total_point: 'Maximum Points', ham: 3143, sch: 1180 },
        { total_race: 'Total Races', ham: 135, sch: 118 },
        { total_race: 'Total Races', ham: 135, sch: 118 },
        { total_race: 'Total Races', ham: 135, sch: 118 },
        { total_race: 'Total Races', ham: 135, sch: 118 }
    ];

    let container;
    let svg;
    let isVisible;
    let tooltip;
    let allInitialized = false;
    let bar1;
    let bar2;
    let xScale;
    const barHeight = 37;

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
            // Reset initial state
            bar1.attr('x', xScale(0)+70)
                .attr('width', 0);

            bar2.attr('x', xScale(0)-70)
                .attr('width', 0);

            // Animate to visible state
            bar1.transition()
                .duration(1000) // Duration of the transition
                .attr('width', d => Math.abs(xScale(d.ham) - xScale(0))) // Animate to final width
                .on('end', function () { // Once the transition is complete, attach event listeners
                    d3.select(this)
                        .on('mouseover', (event, d) => handleBarMouseOver(event, d, 'ham'))
                        .on('mouseout', (event, d) => handleBarMouseOut(event, d, 'ham'))
                        .on('click', (event, d) => handleBarMouseClick(event, d, 'ham'));
                });

            bar2.transition()
                .duration(1000) // Duration of the transition
                .attr('x', d => xScale(0) - Math.abs(xScale(-d.sch) - xScale(0)) -70) // Move bar2 to the left
                .attr('width', d => Math.abs(xScale(-d.sch) - xScale(0))) // Animate to final width
                .on('end', function () { // Once the transition is complete, attach event listeners
                    d3.select(this)
                        .on('mouseover', (event, d) => handleBarMouseOver(event, d, 'sch'))
                        .on('mouseout', (event, d) => handleBarMouseOut(event, d, 'sch'))
                        .on('click', (event, d) => handleBarMouseClick(event, d, 'sch'));
                });
        } else {
            // Animate back to width 0
            bar1.transition()
                .duration(1000) // Duration of the transition
                .attr('width', 0)
                .on('end', function () { // Once the transition is complete, remove event listeners
                    d3.select(this).on('mouseover', null).on('mouseout', null).on('click', null);
                });

            bar2.transition()
                .duration(1000) // Duration of the transition
                .attr('x', xScale(0) - 70) // Move bar2 back to the center
                .attr('width', 0)
                .on('end', function () { // Once the transition is complete, remove event listeners
                    d3.select(this).on('mouseover', null).on('mouseout', null).on('click', null);
                });
        }
    }

    function handleBarMouseOver(event, d, key) {
        d3.select(event.currentTarget).attr('fill', '#ff9729');
        d3.select(event.currentTarget.parentNode)
            .append('text')
            .attr('class', 'highlighted-text')
            .attr('x', +event.currentTarget.getAttribute('x') + +event.currentTarget.getAttribute('width')/2)
            .attr('y', +event.currentTarget.getAttribute('y') + +event.currentTarget.getAttribute('height')/2)
            .attr('dy', '.35em')
            .attr('text-anchor', 'middle')
            .attr('fill', 'black')
            .text(`${d3.format(".2f")(d[key])}%`);
    }

    function handleBarMouseOut(event, d, key) {
        if(key=='ham'){
            d3.select(event.currentTarget).attr('fill', '#5CA4FF');
        }else if(key=='sch'){
            d3.select(event.currentTarget).attr('fill', '#8CC84B');
        }
        d3.select(event.currentTarget.parentNode).selectAll('.highlighted-text').remove();
    }

    function handleBarMouseClick(event, d, key) {
        d3.select('#bar-annotation-2').selectAll('p').remove();
        d3.select('#bar-annotation-2')
            .append('p')
            .style('font-size', '18px')
            .style('padding', '10px 50px')
            .style('text-align', 'left')
            .style('font-weight', '300')
            .text(stats_annotation[Object.keys(d)[0]]);
    }

    onMount(() => {
        const svgWidth = width * 0.75;
        const svgHeight = height * 0.7;
        const margin = { top: 10, right: 20, bottom: 10, left: 10 };
        const widthPlsChart = svgWidth - margin.left - margin.right;
        const heightPlsChart = svgHeight * 0.75 - margin.top - margin.bottom;
        const extraPadding = 70; 

        container = d3.select('#chart-container-2');

        // Remove any existing SVG elements inside the container
        container.selectAll('svg').remove();

        // Append a new SVG with the specified dimensions
        svg = container.append('svg')
            .attr('width', widthPlsChart + margin.left + margin.right)
            .attr('height', heightPlsChart + margin.top + margin.bottom)
            .append('g')
            .attr('transform', `translate(${margin.left},${margin.top})`);

        // Scales
        const maxDataValue = d3.max(per_win_stats, d => Math.max(d.ham, d.sch));
        xScale = d3.scaleLinear()
            .domain([-maxDataValue - extraPadding, maxDataValue + extraPadding])
            .range([0, widthPlsChart]);

        const yScale = d3.scaleBand()
            .domain(per_win_stats.map(d => Object.keys(d)[0]))
            .range([0, heightPlsChart])
            .padding(0.1);

        tooltip = d3.select("body").append("div")
            .attr("class", "tooltip")
            .style("opacity", 0);
        // Calculate the space between bar1 and bar2 for labelAttr
        const spaceBetweenBars = 70;

        // Draw bars
        bar1 = svg.selectAll('.bar1')
            .data(per_win_stats)
            .enter().append('rect')
            .attr('class', 'bar1')
            .attr('x', d => xScale(Math.min(0, d.ham)) + spaceBetweenBars)
            .attr('y', d => yScale(Object.keys(d)[0]))
            .attr('width', 0)
            .attr('height', barHeight)
            .attr('fill', '#5CA4FF');

        // Add labels for bar1
        svg.selectAll('.label1')
            .data(per_win_stats)
            .enter().append('text')
            .attr('class', 'label1')
            .attr('x', widthPlsChart - 10)
            .attr('y', (d, i) => yScale(Object.keys(d)[0]) + 7)
            .attr('dy', '.35em')
            .attr('fill', "#FAEF5D")
            .attr('cursor', 'pointer')
            .attr('text-anchor', 'end')
            .each(function (d, i) {
                const label = d3.select(this);
                const xPosition = widthPlsChart - 10; 
                if (Object.values(per_win_stats[i])[1] >= Object.values(per_win_stats[i])[2]) {
                    label.append('tspan')
                        .attr('x', xPosition) // Set the x position
                        .text(` ${Object.values(win_stats[i])[0]}: ${Object.values(win_stats[i])[1]}`)
                        .attr('fill', '#ff9729')
                        .style('font-weight', 'bold');
                        
                    label.append('tspan')
                        .attr('x', xPosition + 30)
                        .attr('dy', '1em')
                        .html('<tspan>&#x1F3C6;</tspan>');
            
                    label.append('tspan')
                        .attr('x', xPosition) // Set the x position
                        .attr('dy', '0.5em') // Adjust the y-coordinate for the second line
                        .text(` ${Object.values(total_win_stats[i])[0]}: ${Object.values(total_win_stats[i])[1]}`)
                        .attr('fill', '#ff9729')
                        .style('font-size', 'smaller');

                } else {
                    label.append('tspan')
                        .attr('x', xPosition)
                        .text(` ${Object.values(win_stats[i])[0]}: ${Object.values(win_stats[i])[1]}`)
                        .attr('fill', '#5CA4FF')
                        .style('font-weight', 'bold');

                    label.append('tspan')
                        .attr('x', xPosition)
                        .attr('dy', '1.8em') // Adjust the y-coordinate for the second line
                        .text(` ${Object.values(total_win_stats[i])[0]}: ${Object.values(total_win_stats[i])[1]}`)
                        .attr('fill', '#5CA4FF')
                        .style('font-size', 'smaller');
                }
            });


        // Add labels between bar1 and bar2
        svg.selectAll('.labelAttr')
            .data(per_win_stats)
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
            .data(per_win_stats)
            .enter().append('rect')
            .attr('class', 'bar2')
            .attr('x', d => xScale(Math.min(0, -d.sch)) - spaceBetweenBars)
            .attr('y', d => yScale(Object.keys(d)[0]))
            .attr('width', 0)
            .attr('height', barHeight)
            .attr('fill', '#8CC84B');
        
        // Add labels for bar2
        svg.selectAll('.label2')
            .data(per_win_stats)
            .enter().append('text')
            .attr('class', 'label2')
            .attr('x', margin.left + 30)
            .attr('y', (d, i) => yScale(Object.keys(d)[0]) + barHeight / 2 - 13)
            .attr('dy', '.35em')
            .attr('fill', "#FAEF5D")
            .attr('cursor', 'pointer')
            .each(function (d, i) {
                const label = d3.select(this);
                const xPosition = margin.left + 30; 
                if (Object.values(per_win_stats[i])[2] >= Object.values(per_win_stats[i])[1]) {
                    label.append('tspan')
                        .attr('x', xPosition) // Set the x position
                        .text(` ${Object.values(win_stats[i])[0]}: ${Object.values(win_stats[i])[2]}`)
                        .attr('fill', '#ff9729')
                        .style('font-weight', 'bold');
                        
                    label.append('tspan')
                        .attr('x', xPosition - 30)
                        .attr('dy', '1em')
                        .html('<tspan>&#x1F3C6;</tspan>');
            
                    label.append('tspan')
                        .attr('x', xPosition) // Set the x position
                        .attr('dy', '0.5em') // Adjust the y-coordinate for the second line
                        .text(` ${Object.values(total_win_stats[i])[0]}: ${Object.values(total_win_stats[i])[2]}`)
                        .attr('fill', '#ff9729')
                        .style('font-size', 'smaller');

                } else {
                    label.append('tspan')
                        .attr('x', xPosition)
                        .text(` ${Object.values(win_stats[i])[0]}: ${Object.values(win_stats[i])[2]}`)
                        .attr('fill', '#8CC84B')
                        .style('font-weight', 'bold');

                    label.append('tspan')
                        .attr('x', xPosition)
                        .attr('dy', '1.8em') // Adjust the y-coordinate for the second line
                        .attr('fill', '#8CC84B')
                        .text(` ${Object.values(total_win_stats[i])[0]}: ${Object.values(total_win_stats[i])[2]}`)
                        .style('font-size', 'smaller');
                }
            });
    
        const scaleRight = d3.scaleLinear()
            .domain([maxDataValue + extraPadding, 0])
            .range([-spaceBetweenBars * 2 + margin.left * 2, widthPlsChart/2 - spaceBetweenBars]);

        const xAxis1 = svg.append('g')
            .attr('class', 'x-axis')
            .attr('transform', 'translate(0,' + (heightPlsChart) + ')')
            .call(d3.axisTop(scaleRight).tickValues([0, 20, 40, 60, 80])
            .tickFormat(d => (d <= 100) ? (d + '%') : null))
            .selectAll('text')  // Select all tick text elements
            .attr('dx', '0.5em'); 

        // Customize the tick labels appearance
        xAxis1.selectAll('text')
            .style('text-anchor', 'end')
            .attr('dx', '-.8em')
            .attr('dy', '.15em');

        const scaleLeft = d3.scaleLinear()
            .domain([0, maxDataValue + extraPadding])
            .range([widthPlsChart / 2 + spaceBetweenBars, widthPlsChart+ 2*spaceBetweenBars - margin.right * 2]);

        const xAxis2 = svg.append('g')
            .attr('class', 'x-axis')
            .attr('transform', 'translate(0,' + (heightPlsChart) + ')')
            .call(d3.axisTop(scaleLeft).tickValues([0, 20, 40, 60, 80])
            .tickFormat(d => (d <= 100) ? (d + '%') : null))
            .selectAll('text')  // Select all tick text elements
            .attr('dx', '0.5em'); 

        // Customize the tick labels appearance
        xAxis2.selectAll('text')
            .style('text-anchor', 'end')
            .attr('dx', '-.8em')
            .attr('dy', '.15em');
        
        allInitialized = true;
    });
</script>

<div id="chart-container-2" class:visible={isVisible}></div>
<div id='bar-annotation-2'><p style='color:white'>Click on any bar to show more information.</p></div>

<style>
     #chart-container-2{
        opacity: 0;
        visibility: hidden;
        transition: opacity 2s, visibility 2s;
    }
    #chart-container-2.visible{
        opacity: 1;
        visibility: visible;
    }
</style>