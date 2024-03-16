<script>
    import * as d3 from 'd3';
    import { onMount } from 'svelte';

    export let index, width, height;

    // Updated Data
    const per_career_stats = [
        { percentage_points: 'Points', ham: parseFloat((0.598955647 * 100).toFixed(2)), sch: parseFloat((0.379176755 * 100).toFixed(2)) },
        { percentage_wins: 'Wins', ham: parseFloat((0.309309309 * 100).toFixed(2)), sch: parseFloat((0.297385621 * 100).toFixed(2)) },
        { percentage_fl: 'Fastest Laps', ham: parseFloat((0.195195195 * 100).toFixed(2)), sch: parseFloat((0.251633987 * 100).toFixed(2)) },
        { percentage_pod: 'Podiums', ham: parseFloat((0.591591592 * 100).toFixed(2)), sch: parseFloat((0.506535948 * 100).toFixed(2)) },
        { percentage_poles: 'Poles', ham: parseFloat((0.312312312 * 100).toFixed(2)), sch: parseFloat((0.222222222 * 100).toFixed(2)) }
    ];

    const stats_annotation = {
        percentage_points: 'The difference in the maximum points for the two drivers is due to the different number of races they have participated in, and also because of changes in the points scoring system. The points shown above are the actual points they obtained through different scoring systems. If all of their race results were calculated using the most recent scoring metrics, then Schumacher would earn 3961 out of 7956 possible points (49.79%), averaging 12.94 points per race, and Hamilton would earn 5047.5 out of 8754 possible points (57.66%), averaging 15 points per race.',
        percentage_wins: 'The win percentage between the two drivers are extremely close. But because Hamilton is still a current driver, his percentage win would decrease with more participating races. By the end of 2024 season, he would have a lower win rate than Schumacher, assuming he would not get another race win. ',
        percentage_fl: "The amount of fastest laps reflects a driver's consistently fast race pace throughout his career. Schumacher holds the most amount of fastest laps records among all F1 drivers, meaning that he was able to out perform all other drivers on the grid at each grand prix. ",
        percentage_pod: "If a driver consistently finished on the podium, this often implied that the car was reliable during the race, and the team's race car is quite competitive, standing in a position ready to take victories. In a way, the amount of podiums also shows how dominant a driver or team is in that season. Both drivers finished on the podium in MORE THAN HALF of the races through their career. ",
        percentage_poles: "The amount of poles could reflect a driver's one-lap pace. But if we were to look at the pole to win ratio (poles/wins), we see that Schumacher has a ratio of 0.75 meaning that more than 1/4 th of Schumacher's wins comes from him overtaking in the race and not purely relying on the pace of the car.  While Hamilton has a ratio of 1.01, meaning that a large majority of the wins (if not close to all), comes from his pole position starts, meaning that his wins might have relied more on the pace of the car. "
    }

    const career_stats = [
        { points: '#Points', ham: 4646, sch: 1566 },
        { wins: '#Wins', ham: 103, sch: 91 },
        { fl: '#Fastest Laps', ham: 65, sch: 77 },
        { pod: '#Podiums', ham: 197, sch: 155 },
        { poles: '#Poles', ham: 104, sch: 68 }
    ];

    const total_career_stats = [
        { total_point: 'Maximum Points', ham: 7756, sch: 4130 },
        { total_race: 'Total Races', ham: 333, sch: 306 },
        { total_race: 'Total Races', ham: 333, sch: 306 },
        { total_race: 'Total Races', ham: 333, sch: 306 },
        { total_race: 'Total Races', ham: 333, sch: 306 }
    ];

    let container;
    let svg;
    let tooltip;
    let isVisible;
    let allInitialized = false;
    let bar1;
    let bar2;
    const barHeight = 37;
    let xScale;

    
    $:if(index == 2){
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
        d3.select('#bar-annotation-1').selectAll('p').remove();
        d3.select('#bar-annotation-1')
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
        container = d3.select('#chart-container-1');
        // Remove any existing SVG elements inside the container
        container.selectAll('svg').remove();

        // Append a new SVG with the specified dimensions
        svg = container.append('svg')
            .attr('width', widthPlsChart + margin.left + margin.right)
            .attr('height', heightPlsChart + margin.top + margin.bottom)
            .append('g')
            .attr('transform', `translate(${margin.left},${margin.top})`);

        // Scales
        const maxDataValue = d3.max(per_career_stats, d => Math.max(d.ham, d.sch));
        xScale = d3.scaleLinear()
            .domain([-maxDataValue - extraPadding, maxDataValue + extraPadding])
            .range([0, widthPlsChart]);

        const yScale = d3.scaleBand()
            .domain(per_career_stats.map(d => Object.keys(d)[0]))
            .range([0, heightPlsChart])
            .padding(0.1);

        tooltip = d3.select("body").append("div")
            .attr("class", "tooltip")
            .style("opacity", 0);
        // Calculate the space between bar1 and bar2 for labelAttr
        const spaceBetweenBars = 70;

        // Draw bars
        bar1 = svg.selectAll('.bar1')
            .data(per_career_stats)
            .enter().append('rect')
            .attr('class', 'bar1')
            .attr('x', d => xScale(Math.min(0, d.ham)) + spaceBetweenBars)
            .attr('y', d => yScale(Object.keys(d)[0]))
            .attr('width', 0)
            .attr('height', barHeight)
            .attr('fill', '#5CA4FF');

        // Add labels for bar1
        svg.selectAll('.label1')
            .data(per_career_stats)
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
                if (Object.values(per_career_stats[i])[1] >= Object.values(per_career_stats[i])[2]) {
                    label.append('tspan')
                        .attr('x', xPosition) // Set the x position
                        .text(` ${Object.values(career_stats[i])[0]}: ${Object.values(career_stats[i])[1]}`)
                        .attr('fill', '#ff9729')
                        .style('font-weight', 'bold');
                        
                    label.append('tspan')
                        .attr('x', xPosition + 30)
                        .attr('dy', '1em')
                        .html('<tspan>&#x1F3C6;</tspan>');
            
                    label.append('tspan')
                        .attr('x', xPosition) // Set the x position
                        .attr('dy', '0.5em') // Adjust the y-coordinate for the second line
                        .text(` ${Object.values(total_career_stats[i])[0]}: ${Object.values(total_career_stats[i])[1]}`)
                        .attr('fill', '#ff9729')
                        .style('font-size', 'smaller');

                } else {
                    label.append('tspan')
                        .attr('x', xPosition)
                        .text(` ${Object.values(career_stats[i])[0]}: ${Object.values(career_stats[i])[1]}`)
                        .attr('fill', '#5CA4FF')
                        .style('font-weight', 'bold');

                    label.append('tspan')
                        .attr('x', xPosition)
                        .attr('dy', '1.8em') // Adjust the y-coordinate for the second line
                        .text(` ${Object.values(total_career_stats[i])[0]}: ${Object.values(total_career_stats[i])[1]}`)
                        .attr('fill', '#5CA4FF')
                        .style('font-size', 'smaller');
                }
            });

            
        // Add labels between bar1 and bar2
        svg.selectAll('.labelAttr')
            .data(per_career_stats)
            .enter().append('text')
            .attr('class', 'labelAttr')
            .attr('x', widthPlsChart/2) // Centered position
            .attr('y', d => yScale(Object.keys(d)[0]) + barHeight / 2)
            .attr('dy', '.35em')
            .attr('text-anchor', 'middle') // Center the text
            .attr('fill', "#FAEF5D")
            .style('font-size', '1.2em') 
            .text(d => Object.values(d)[0]);

        // Draw bars
        bar2 = svg.selectAll('.bar2')
            .data(per_career_stats)
            .enter().append('rect')
            .attr('class', 'bar2')
            .attr('x', d => xScale(Math.min(0, -d.sch)) - spaceBetweenBars)
            .attr('y', d => yScale(Object.keys(d)[0]))
            .attr('width', 0)
            .attr('height', barHeight)
            .attr('fill', '#8CC84B');
        

        // Add labels for bar2
        svg.selectAll('.label2')
            .data(per_career_stats)
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
                if (Object.values(per_career_stats[i])[2] >= Object.values(per_career_stats[i])[1]) {
                    label.append('tspan')
                        .attr('x', xPosition) // Set the x position
                        .text(` ${Object.values(career_stats[i])[0]}: ${Object.values(career_stats[i])[2]}`)
                        .attr('fill', '#ff9729')
                        .style('font-weight', 'bold');
                        
                    label.append('tspan')
                        .attr('x', xPosition - 30)
                        .attr('dy', '1em')
                        .html('<tspan>&#x1F3C6;</tspan>');
            
                    label.append('tspan')
                        .attr('x', xPosition) // Set the x position
                        .attr('dy', '0.5em') // Adjust the y-coordinate for the second line
                        .text(` ${Object.values(total_career_stats[i])[0]}: ${Object.values(total_career_stats[i])[2]}`)
                        .attr('fill', '#ff9729')
                        .style('font-size', 'smaller');

                } else {
                    label.append('tspan')
                        .attr('x', xPosition)
                        .text(` ${Object.values(career_stats[i])[0]}: ${Object.values(career_stats[i])[2]}`)
                        .attr('fill', '#8CC84B')
                        .style('font-weight', 'bold');

                    label.append('tspan')
                        .attr('x', xPosition)
                        .attr('dy', '1.8em') // Adjust the y-coordinate for the second line
                        .attr('fill', '#8CC84B')
                        .text(` ${Object.values(total_career_stats[i])[0]}: ${Object.values(total_career_stats[i])[2]}`)
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

<div id="chart-container-1" class:visible={isVisible}></div>
<div id='bar-annotation-1'><p style='color:white'>Click on any bar to show more information.</p></div>

<style>
    #chart-container-1{
        opacity: 0;
        visibility: hidden;
        transition: opacity 2s, visibility 2s;

    }
    #chart-container-1.visible{
        opacity: 1;
        visibility: visible;
    }

    #chart-container-1 {
        fill:aquamarine;
    }
</style>