<script>
    import * as d3 from 'd3';
    import { onMount } from 'svelte';

    export let index, width, height;
    const per_dom_stats = [
        { percentage_points: 'Points', ham: parseFloat((0.785067873 * 100).toFixed(2)), sch: parseFloat((0.822222222 * 100).toFixed(2)) },
        { percentage_wins: 'Wins', ham: parseFloat((0.647058824 * 100).toFixed(2)), sch: parseFloat((0.722222222 * 100).toFixed(2)) },
        { percentage_fl: 'Fastest Laps', ham: parseFloat((0.352941176 * 100).toFixed(2)), sch: parseFloat((0.555555556 * 100).toFixed(2)) },
        { percentage_pod: 'Podiums', ham: parseFloat((0.823529412 * 100).toFixed(2)), sch: parseFloat((0.833333333 * 100).toFixed(2)) },
        { percentage_poles: 'Poles', ham: parseFloat((0.588235294 * 100).toFixed(2)), sch: parseFloat((0.444444444 * 100).toFixed(2)) }
        ];

    const stats_annotation = {
        percentage_points: "Both drivers get astonishingly close to earning the maximum amount of points possible, this would require no mishaps from the team strategy, while the driver has to showcase absolutely impeccable driving, and almost perfect performance. However, Schumacher was even more domiant than Hamilton, achieving 2 grand slams, meaning taking pole position, leading every lap of the race, finishing first place, while holding the fastest lap record. ",
        percentage_wins: "Even though the difference in win rates between the drivers seem huge, Hamilton had only missed one additional win in comparison to Schumacher, with Schumacher missing 5 race wins, and Hamilton missing 6. One of the misses is even due to Hamilton's suboptimal health condition for catching COVID-19. However, the Ferrari driver was able to achieve a win streak of 7, while the Mercedes driver could only achieve 5. ",
        percentage_fl: "One reason that Hamilton consistently had fewer fastest laps than Schumacher might have been due to the strategy employed by Mercedes. Most of the time, in order to get the fastest lap of the race, a driver would need to swap on new soft rubber, but that would also sacrafice overall race time as it requires an additional pitstop. The tyre characteristics at Schumacher's time was different, and didn't have as much degredation as current tyres, so he wouldn't have needed to stop for new ones before going for the fastest lap. ",
        percentage_pod: "Both their number of podiums were extremely close, meaning that they were likely equally as consistent, driving similarly dominant and reliable machines, hurling around the tracks. It is quite hard to tell which driver-team combo achieved better results simply by looking at these close figures.",
        percentage_poles: "One reason why Schumacher didn't manage to get as high of a pole percentage as Hamilton is due to strategy choices. Ferrari at the time believed that sometimes by trading pole position, drivers could achieve a better start to the race and are set up for better overtakes. ",
    }

    const dom_stats = [
        { points: 'Points Count', ham: 347, sch: 148 },
        { wins: 'Wins Count', ham: 11, sch: 13 },
        { fl: 'Fastest Laps Count', ham: 6, sch: 10 },
        { pod: 'Podiums Count', ham: 14, sch: 15 },
        { poles: 'Poles Count', ham: 10, sch: 8 }
    ];

    const total_dom_stats = [
        { total_point: 'Maximum Points', ham: 442, sch: 180 },
        { total_race: 'Total Races', ham: 17, sch: 18 },
        { total_race: 'Total Races', ham: 17, sch: 18 },
        { total_race: 'Total Races', ham: 17, sch: 18 },
        { total_race: 'Total Races', ham: 17, sch: 18 }
    ];

    let container;
    let svg;
    let isVisible;
    let tooltip;
    let allInitialized = false;
    let bar1;
    let bar2;
    let xScale;
    const barHeight =35;

    $:if(index == 4){
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
        d3.select('#bar-annotation-3').selectAll('p').remove();
        d3.select('#bar-annotation-3')
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
            .attr('x', widthPlsChart - 10)
            .attr('y', (d, i) => yScale(Object.keys(d)[0]) + 7)
            .attr('dy', '.35em')
            .attr('fill', "#FAEF5D")
            .attr('cursor', 'pointer')
            .attr('text-anchor', 'end')
            .each(function (d, i) {
                const label = d3.select(this);
                const xPosition = widthPlsChart - 10; 
                if (Object.values(per_dom_stats[i])[1] >= Object.values(per_dom_stats[i])[2]) {
                    label.append('tspan')
                        .attr('x', xPosition) // Set the x position
                        .text(` ${Object.values(dom_stats[i])[0]}: ${Object.values(dom_stats[i])[1]}`)
                        .attr('fill', '#ff9729')
                        .style('font-weight', 'bold');
                        
                    label.append('tspan')
                        .attr('x', xPosition + 30)
                        .attr('dy', '1em')
                        .html('<tspan>&#x1F3C6;</tspan>');
            
                    label.append('tspan')
                        .attr('x', xPosition) // Set the x position
                        .attr('dy', '0.5em') // Adjust the y-coordinate for the second line
                        .text(` ${Object.values(total_dom_stats[i])[0]}: ${Object.values(total_dom_stats[i])[1]}`)
                        .attr('fill', '#ff9729')
                        .style('font-size', 'smaller');

                } else {
                    label.append('tspan')
                        .attr('x', xPosition)
                        .text(` ${Object.values(dom_stats[i])[0]}: ${Object.values(dom_stats[i])[1]}`)
                        .attr('fill', '#5CA4FF')
                        .style('font-weight', 'bold');

                    label.append('tspan')
                        .attr('x', xPosition)
                        .attr('dy', '1.8em') // Adjust the y-coordinate for the second line
                        .text(` ${Object.values(total_dom_stats[i])[0]}: ${Object.values(total_dom_stats[i])[1]}`)
                        .attr('fill', '#5CA4FF')
                        .style('font-size', 'smaller');
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
            .attr('y', (d, i) => yScale(Object.keys(d)[0]) + barHeight / 2 - 13)
            .attr('dy', '.35em')
            .attr('fill', "#FAEF5D")
            .attr('cursor', 'pointer')
            .each(function (d, i) {
                const label = d3.select(this);
                const xPosition = margin.left + 30; 
                if (Object.values(per_dom_stats[i])[2] >= Object.values(per_dom_stats[i])[1]) {
                    label.append('tspan')
                        .attr('x', xPosition) // Set the x position
                        .text(` ${Object.values(dom_stats[i])[0]}: ${Object.values(dom_stats[i])[2]}`)
                        .attr('fill', '#ff9729')
                        .style('font-weight', 'bold');
                        
                    label.append('tspan')
                        .attr('x', xPosition - 30)
                        .attr('dy', '1em')
                        .html('<tspan>&#x1F3C6;</tspan>');
            
                    label.append('tspan')
                        .attr('x', xPosition) // Set the x position
                        .attr('dy', '0.5em') // Adjust the y-coordinate for the second line
                        .text(` ${Object.values(total_dom_stats[i])[0]}: ${Object.values(total_dom_stats[i])[2]}`)
                        .attr('fill', '#ff9729')
                        .style('font-size', 'smaller');

                } else {
                    label.append('tspan')
                        .attr('x', xPosition)
                        .text(` ${Object.values(dom_stats[i])[0]}: ${Object.values(dom_stats[i])[2]}`)
                        .attr('fill', '#8CC84B')
                        .style('font-weight', 'bold');

                    label.append('tspan')
                        .attr('x', xPosition)
                        .attr('dy', '1.8em') // Adjust the y-coordinate for the second line
                        .attr('fill', '#8CC84B')
                        .text(` ${Object.values(total_dom_stats[i])[0]}: ${Object.values(total_dom_stats[i])[2]}`)
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

<div id="chart-container-3" class:visible={isVisible}></div>
<div id='bar-annotation-3'><p style='color:white'>Click on any bar to show more information.</p></div>

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