<script>
  import Scroller from "@sveltejs/svelte-scroller";
  import EachYear from "./EachYear.svelte";
  import Route from "./CarRoute.svelte";
  import Silverstone from "./Silverstone.svelte";
  import LapMovement from "./LapMovement.svelte";
  import DriverCareer from "./DriverCareer.svelte";
  import DriverWin from "./DriverWinningYear.svelte";
  import DriverDom from "./DriverDominant.svelte";
  import Timeline from "./Timeline.svelte";
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
    <!-- background elements -->
  </div>

  <div class="foreground" slot="foreground">
    <section class='titlescreen'>
      <div class='behind'></div>
      <div class='infront'>
        <span class="main-title">Is Hamilton More Dominant Than Schumacher?</span><br>
        <span>Battle for the GOAT</span>
        <div class="content">
            <p>A comparison of race statistics for the two Formula 1 World Champions</p>
        </div>
        <div class="route">
        </div>
        <!-- svelte-ignore a11y-img-redundant-alt -->
        <img src="https://raw.githubusercontent.com/EskimoSun/F1_statistics_analysis/main/assets/photo2.png" alt="Comparison Photo" width="900" height="500"/>
      </div>
    </section>

    <section class='intro-page'>
      <Route />
      <div class='introduction'>
        <p>
          Both Lewis Hamilton and Michael Schumacher are towering figures in the world of Formula 1 racing and world champions. One being the first and only black driver in the sports to achieve a height that no other person had, and the other being synonymous with Ferrari’s success, laying down unbeatened records in the early 2000s.   
        </p>
        <br>
        <span style='font-weight:bold;font-size:25px;text-align:center'>But can we really tell who is the better driver?</span>
        <br>
        <br>
        <p>
          In Formula 1, in order to become a world champion, you would have to earn enough racing points and be on top of the leaderboard at the end of a season. By winning each round of grand prix races, or finishing in a high positon, you would get the crucial victory points. Both Michael Schumacher and Lewis Hamilton share the record of 7 world championship titles, which would cement their legendary status in the motorsports hall of fame.
        </p>
        <p>
          Lewis Hamilton's F1 career started in 2007 and immediately started collecting race wins left, right, and center, even when he was facing head-to-head against his already-accomplished, world champion teammate. But he only started dominating the sports in 2014, when had partnered with Mercedes as his team, even till today.
        </p>
        <p>
          Michael Schumacher's name reigns as one of the most famous drivers anyone has ever heard. His debut in 1991 as a F1 driver was mostly unexpected, but through serious dedication and fierce pace, he earned himself a reputation of strength and tenacity. His cooperation with Ferrari from 1996 to 2006 has secured him a racing legacy well respected by fans everywhere. In his dominance, no other driver could get close to the speed that he exhibited. 
        </p>
      </div>
      <Timeline/>
    </section>

    <section style="height:105vh">
      <p class="chart-title">Career Stats</p>
      <p style="font-size: 22px;">What is Hamilton and Schumacher Overall Career Achievements?</p>
      <div class="bottom-section">
        <div class="bottom-left">
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

    <section style="height:107vh">
      <p class="chart-title">Championship Winning Season</p>
      <p style="font-size: 18px;padding: 0px 80px;text-align: left">Sometimes looking at the overall career achievements doesn't tell the full story. Both drivers had mediocre seasons when their team's car simply lacked the speed to rank at the front of the grid. Luckily, both drivers had 7 phenomenal seasons when they had won the championship. Lets evaluate the drivers' performance at their prime!</p>
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

    <section style="height:120vh;">
      <p class="chart-title">Most Dominant Season (MSC: 2004, HAM: 2020)</p>
      <p style="font-size: 20px;">What if we narrow down even further and compare their absolute peak dominance?</p>
      <div class="bottom-section">
        <div class="bottom-left">
          <!-- Add your text content here -->
          <br> 
          <br> 
          <br>
          <p style="text-align: left; font-size: 18px;">When boiling down to their absolute most dominant season, Schumacher shows <span style="color: #ff9729; font-weight: bold; font-size: 22px">an overwhelming overtake</span> in most statistics.</p>
          <br>
          <br>
          <p style="text-align: left; font-size: 18px;">But the details on whichever driver is the greater driver is ambiguous. Discussed in our additional information, many factors such as points system, strategy, and tyre plays such an effect on the statistics of the driver that by simply looking at the numbers <span style="color: #ff9729; font-weight: bold; font-size: 22px"> couldn't be conclusive. </span></p>
          <br>
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
      <p style="text-align: left; font-size: 18px; margin: 50px 70px 20px 50px"><span style="color: #ff9729; font-weight: bold; font-size: 22px">Therefore</span>,  one simple glance at the statistics of any two sports champion doesn't tell who is the greater player. You would have to look at the implications behind specific characteristics within their own time frames like we have shown in our bar chart, and discover why they are great in their own way.</p>
      <p style="text-align: center; font-size: 18px; font-weight:bold;">Continue below and see how your favorite driver stacks up against Schumacher and Hamilton!</p>
    </section>

    <section>
      <p class="chart-title">Total Points From Each Driver By Season</p>
      <EachYear {index} {width} {height}/>
    </section>

    <section style="height:130vh">
      <p class="chart-title">Laptime Difference To Leading Driver</p>
      <LapMovement {index} {width} {height}/>
      <p style="color: aliceblue;">Page created by Zhiqing Wang & David Sun for DSC 106</p>
    </section>

  </div>
</Scroller>

<style>
  .background {
    width: 100%;
    height: 100vh;
    position: relative;
    outline: grey solid 3px;
    background-color: #0D1B43;
  }

  .foreground {
    width: 100%;
    margin: 0 auto;
    height: auto;
    position: relative;

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

  section {
    height: 110vh;
    background-color: #0D1B43;
    /* color: white; */
    outline: grey solid 1px;
    text-align: center;
    max-width: 1500px; /* adjust at will */
    color: #FAEF5D;
    padding: 1em;
    margin: 0 0 2em 0;
  }

  section.titlescreen{
    font-size: 25px;
    text-shadow: 2px 2px 4px #000000;
    height: 100vh;
  }

  .chart-title {
    font-size: 16px;
    font-weight: bold;
  }

  .main-title{
    font-size: 50px;
    font-weight: bold;
  }

  .bottom-section {
    display: flex;
    width: 100%;
    margin-top: -30px;
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

  .introduction{
    font-size: 20px;
    margin: 0 auto;
    padding: 5px 40px;
    width: 90%;
    text-align: left;
    background-color: #2D3B63;
  }

  .intro-page{
    height:155vh;
    overflow: hidden;
  }

  .behind{
    z-index: 1;
    position: absolute;
    background: url('https://images7.alphacoders.com/120/1201384.jpg'); /* Background image source*/
    width: 99vw;
    height: 107.5vh;
    margin-left: -25px;
    margin-top: -25px;
    background-repeat: no-repeat;
    background-size: cover;
    opacity: 0.5;
  }
  .infront{
    z-index: 2;
    position: relative;
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

