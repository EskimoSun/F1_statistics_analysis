<script>
  import Scroller from "@sveltejs/svelte-scroller";
  import EachYear from "./EachYear.svelte";
  import LaptimeDiff from "./LaptimeDiff.svelte";
  import Silverstone from "./Silverstone.svelte";
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
      <span class="title">Is Verstappen More Dominant Than Schumacher?</span>
      <div>
        <span>Battle for the GOAT</span>
        <p>A comparison of race statistics for the two Formula 1 World Champions</p>
        <Silverstone />
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
    outline: grey solid 3px;
    background-color: #1D2B53;
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
    outline: grey solid 3px;
    text-align: center;
    max-width: 1500px; /* adjust at will */
    color: #FAEF5D;
    padding: 1em;
    margin: 0 0 2em 0;
  }

  span.title{
    font-size: 50px;

  }
  section.title{
    height: 50vh;
    font-size: 25px;
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
</style>

