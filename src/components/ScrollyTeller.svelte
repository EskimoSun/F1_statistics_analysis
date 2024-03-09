<script>
  import Scroller from "@sveltejs/svelte-scroller";
  import EachYear from "./EachYear.svelte";
  import LaptimeDiff from "./LaptimeDiff.svelte";
  import Silverstone from "./Silverstone.svelte";
  import DriverCareer from "./DriverCareer.svelte";
  import DriverWin from "./DriverWinningYear.svelte";
  import DriverDom from "./DriverDominant.svelte";
  let count, index, offset, progress;
  let width, height;
</script>

<Scroller
  top={0.0}
  bottom={1}
  threshold={0.5}
  bind:count
  bind:index
  bind:offset
  bind:progress>
  <div class="background" slot="background" bind:clientWidth={width} bind:clientHeight={height}>
    <div class="progress-bars">
      <p>current index: <strong>{index}</strong></p>
      <progress value={count ? (index + 1) / count : 0} />

      <p>offset in current section</p>
      <progress value={offset || 0} />

      <p>total progress</p>
      <progress value={progress || 0} />
    </div>
  </div>


  <div class="foreground" slot="foreground">
    <section class="title">
      <span class="title">Is Hamilton More Dominant Than Schumacher?</span><br>
      <span>Battle for the GOAT</span>
      <div class="content">
          <p>A comparison of race statistics for the two Formula 1 World Champions</p>
      </div>
      <div class="route">
      </div>
      <img src="./src/components/photo2.png" alt="Comparison Photo" width="900" height="500"/>
    </section>
    <section>
      <p class="chart-title main-title">Career Stats</p>
      <p style="font-size: 22px;">What is Hamilton and Schumacher Overall Career Achievements?</p>
      <div class="bottom-section">
        <div class="bottom-left">
          <!-- Add your text content here -->
          <br> 
          <br> 
          <br>
          <p style="text-align: left; font-size: 18px;">Let's look at 5 important attributes to evaluate their performance!</p>
          <p style="text-align: left; font-size: 18px;">Hamilton performs <span style="color: #ff9729; font-weight: bold; font-size: 22px"> overwhelmingly better </span>than Schumacher in almost all aspects in their overall career achievements.</p>
          <br>
          <p style="text-align: left; font-size: 18px;"><span style="color: #ff9729; font-weight: bold; font-size: 22px">But</span> does that make Hamilton the better driver?</p>
          <br>
          <p style="text-align: left; font-size: 15px;">Note. all Bars are represent percentage value, hover it to get more info.</p>
        </div>
        <div class="bottom-right">
          <p class="chart-title subtitle">
            <span class="name first-name">Michael Schumacher</span>
            <span class="name second-name">Lewis Hamilton</span>
          </p>  
          <div class="chart-title chart">
            <DriverCareer {index} {width} {height}/>
          </div>
        </div>
      </div>
    </section>
    <section>
      <p class="chart-title main-title">Championship Winning Season</p>
      <p style="font-size: 22px;">What are Hamilton and Schumacher's Achievements in their 7 Championship Seasons?</p>
      <div class="bottom-section">
        <div class="bottom-left">
          <!-- Add your text content here -->
          <br> 
          <br> 
          <br>
          <p style="text-align: left; font-size: 18px;">Schumacher's lead in <span style="color: #ff9729; font-weight: bold; font-size: 22px">fastest laps</span> and <span style="color: #ff9729; font-weight: bold; font-size: 22px">race wins</span> imply a <span style="color: #ff9729; font-weight: bold; font-size: 22px">quicker race pace</span> than Lewis Hamilton.</p>
          <br>
          <p style="text-align: left; font-size: 18px;">Hamilton's <span style="color: #ff9729; font-weight: bold; font-size: 22px">lower pole-to-win</span> conversion implies that the pace of the car might have helped him get more poles, but he couldn't continue the pace in the race.</p>

          <p style="text-align: left; font-size: 15px;">Schumacher's winning Season: 1994, 1995, 2000, 2001, 2002, 2003, 2004</p>
          <p style="text-align: left; font-size: 15px;">Hamilton's winning Season: 2008, 2014, 2015, 2017, 2018, 2019, 2020</p>
        </div>
        <div class="bottom-right">
          <p class="chart-title subtitle">
            <span class="name first-name">Michael Schumacher</span>
            <span class="name second-name">Lewis Hamilton</span>
          </p>  
          <div class="chart-title chart">
            <DriverWin {index} {width} {height}/>
          </div>
        </div>
      </div>
    </section>
    <section>
      <p class="chart-title main-title">Dominant Season (MSC: 2004, HAM: 2019)</p>
      <p style="font-size: 22px;">What is Hamilton and Schumacher Overall Career Achievements?</p>
      <div class="bottom-section">
        <div class="bottom-left">
          <!-- Add your text content here -->
          <br> 
          <br> 
          <br>
          <p style="text-align: left; font-size: 18px;">When boiling down to their absolute most dominant season, Schumacher shows <span style="color: #ff9729; font-weight: bold; font-size: 22px">a complete overtake</span> in all statistics.</p>
          <p style="text-align: left; font-size: 18px;"><span style="color: #ff9729; font-weight: bold; font-size: 22px"></span></p>
          <br>
          <br>
          <br>
          <br>
          <p style="text-align: left; font-size: 15px;">Note. Keep scroll down to find you favoriate drivers and analyze their performance by yourself!</p>
        </div>
        <div class="bottom-right">
          <p class="chart-title subtitle">
            <span class="name first-name">Michael Schumacher</span>
            <span class="name second-name">Lewis Hamilton</span>
          </p>  
          <div class="chart-title chart">
            <DriverDom {index} {width} {height}/>
          </div>
        </div>
      </div>
    </section>
    <section>
      <p class="chart-title">Total Points From Each Driver By Season</p>
      <EachYear {index} {width} {height}/>
    </section>
    <section>
      <p class="chart-title">Laptime Difference To Leading Driver</p>
      <LaptimeDiff {index} {width} {height}/>
    </section>
    <section class="writeup">
      <span>WRITE UP</span>
      <p>1. What have you done so far?</p>
      <p>For the prototype of the website, we implemented two essential interactive charts that demonstrate an overview of the drivers' statistics and performance throughout a season. For the graphs to work properly, we had to transform our original datasets into json format in order to reduce size. As we designed our interaction, we focused on showing the performance gap between drivers and how these differences accumulated over the year. In addition to the highlighting, we also added selection panels to allow the charts to display historic races, and somewhat compare drivers between different eras.</p>
      <p>2. What will be the most challenging of your project to design and why?</p>
      <p>Our interactions currently don't tell the story that well and is primarily used for exploration purposes. The challenges looking forward would be to implement visual animations that can guide the readers to understand the situation of the drivers, and how they stack up in terms of performance. There will be many custom annotations needed in order to facilitate a clearer demonstration, information such as driver achievements and broken records will have to be noted. For the remainder of the project, we will be directing more of our focus towards creating a compelling story.</p>
    </section>
  </div>
</Scroller>

<style>
  .background {
    width: 100%;
    height: 100vh;
    position: relative;
    outline: green solid 3px;
    background-color: #070F2B;
  }

  .foreground {
    width: 100%;
    margin: 0 auto;
    height: auto;
    position: relative;
    background-color: #0D1B43;

  }

  .title {
    flex: 1;
    width: 100%;
  }

  .content {
    flex: 8;
    align-items: center;
  }

  .route {
    flex: 2;
    align-items: center;
    margin-left: 20px; /* Adjust the margin as needed */
  }

  .progress-bars {
    position: absolute;
    background: rgba(170, 51, 120, 0.2) /*  40% opaque */;
    visibility: visible;
  }
  section {
    height: 100vh;
    background-color: rgba(255, 0, 0, 0.05); /* 20% opaque */
    /* color: white; */
    outline: grey solid 1px;
    text-align: center;
    max-width: 1500px; /* adjust at will */
    color: #FAEF5D;
    padding: 1em;
    margin: 0 0 2em 0;
  }

  section.title{
    height: 80vh;
    font-size: 25px;
  }

  .chart-title {
    font-size: 16px;
  }

  span.title{
    font-size: 50px;

  }

  .main-title {
    font-weight: bold;
  }

  .bottom-section {
    display: flex;
    width: 100%;
    box-sizing: border-box; /* Include padding in the total width */
  }

  .bottom-left {
    width: 20%; 
    padding: 10px; /* Adjusted padding */
  }

  .bottom-right {
    width: 80%; 
    padding: 10px; /* Adjusted padding */
  }

  .bottom-section .bottom-right {
    width: 80%; 
    padding: 10px; /* Adjusted padding */
    position: relative; /* Added to position the names relative to the bottom-right container */
  }

  .subtitle {
    font-size: 14px;
    height: 15%;
    display: flex;
    align-items: center; /* Center names vertically */
  }

  .name {
    font-size: 20px;
    flex: 1; /* Each name takes up equal space in the subtitle */
  }

  .name.first-name {
    color: #8CC84B;
  }

  .name.second-name {
    color: #5CA4FF;
  }

  .chart-title.chart {
    height: 90%;
  }

  section.writeup{
    padding-left: 40px;
    padding-right: 40px;
    font-size: 22px;
  }
  p.chart-title{
    font-size: 25px;
    margin-top: -5px;
    margin-bottom: 12px;
  }
  p {
    margin-bottom: 20px; /* Adjust the margin to increase paragraph spacing */
    line-height: 1.5; /* Adjust the line-height to increase space between lines */
  }
</style>

