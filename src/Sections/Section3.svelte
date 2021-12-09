<script>
  import * as d3 from 'd3';
  import { onMount } from 'svelte';
  import SvelteHeatmap from 'svelte-heatmap'

  export let data;

  const parseTime = d3.timeParse("%Y-%m-%d");
  const formatTime = d3.timeFormat("%B %e");

  data.map(d => {
    d.date = parseTime(d.date);
    d.false = +d.false;
    d.true = +d.true;
    d.pct_ret = +d.pct_ret;
    d.value = +d.pct_ret;
  })

  const intervals = [{
      title: "Warmer months",
      start: parseTime("2018-04-01"),
      end: parseTime("2018-09-30")
    }, {
      title: "Colder months",
      start: parseTime("2018-10-01"),
      end: parseTime("2019-03-31")
    }
  ] 

  const colors = ["#000000", "#393918", "#65652a", "#898920", "#c4c410", "#ffff33"]
  const colors2 = [
    { pct: 8.3, color: "#393918" },
    { pct: 13.1, color: "#65652a" },
    { pct: 17.8, color: "#898920" },
    { pct: 22.6, color: "#c4c410"},
    { pct: 27.3, color: "#ffff33"}
  ]
  let x, y;
  let date, pct;

  onMount(() => {
    d3.selectAll('.heatmap').selectAll('text').attr("transform", "translate(0, 15)").style("fill", "gray")
    const rects = d3.selectAll('.heatmap').selectAll('rect');
    const tooltip = d3.select(".tooltip");
    rects
      .on("mouseover", function(e, d) {
        rects.attr("opacity", 0.3)
        d3.select(this).attr("opacity", 1).attr("stroke-width", 3).attr("stroke", "white")
        console.log(e.target.getBoundingClientRect())
        x = e.layerX + 5
        y = e.layerY + 5
        console.log(e)
        date = d3.select(this).attr("data-date")
        pct = d3.select(this).attr("data-value")
        tooltip.style("top", `${y}px`).style("left", `${x}px`)
          .style('display', 'block')
          .html(`${formatTime(parseTime(date))}<br>
                  <strong>${Math.round(pct * 10) / 10}% in retrograde</strong>`)
      })
      .on("mouseout", function() {
        rects.attr("opacity", 1)
        d3.select(this).attr("opacity", 1).attr("stroke-width", 0)
        tooltip.style('display', 'none')
      })
  })
</script>

<div class="container">
  <h3>What day of the year does mercury retrograde occur the most?</h3>
  <p>
    There have been 37 retrograde periods from 2010 to 2020. Mercury retrograde occers a year round, but there was a pocket of period when Mercury entered retrograde more often, from mid-October to mid-December.
  </p>
  <div class="legend">
    <strong>Percentage of daily retrograde counts from 2010 to 2020</strong>
    <svg width="300" height="45">
      <g>
        {#each colors2 as {pct, color}, i}
          <rect
            x="{i * 50}"
            y="10"
            width="15"
            height="15"
            fill="{color}"
          ></rect>
          <text
            x="{i * 50}"
            y="45"
            fill="gray"
          >{pct}%</text>
        {/each}
      </g>  
    </svg>
  </div>
  <div class="tooltip"></div>
  {#each intervals as i}
    <h4>{i.title}</h4>
    <div class="heatmap">
      <SvelteHeatmap
        allowOverflow={false}
        cellGap={5}
        cellRadius={0}
        colors={colors}
        fontColor={"white"}
        fontSize={12}
        fontFamily={"Fira Sans"}
        data={data}
        dayLabelWidth={0}
        emptyColor={'rgba(0, 0, 30, 0)'}
        startDate={i.start}
        endDate={i.end}
        monthLabelHeight={25}
        view={'yearly'}
      />
    </div>
  {/each}
</div>

<article>
  <p>
    As shown in this heat map, there are pockets of weeks within the past 10 years where retrograde has not transpired as frequently, including January and September. However, flights are delayed on a daily basis including during these dips – sometimes as a result of poor weather conditions. 
  </p>
  <p>
    Flight delays are hectic for the traveler because of issues of communication. Tatiana Fontalvo, an artist interested in astrology, talked about how travels can often be delayed while mercury is in retrograde. She reflected that just by “driving your car or taking a plane, we are communicating and moving to other places. Therefore, [in moments of] retrograde, there is a necessity to be patient, slower and more aware because the energy can be chaotic and unpredictable.” 
  </p>
</article>

<div class="video">
  <video controls width="100%" src="./audio/moon.mp4">
    <track kind="captions">
  </video>
</div>

<article>
  <p>
    “Mercury is the planet of communication, transportation and shipping. And when it sleeps, you start to see the lack of support that the planet usually gives you,” said astrology icon Susan Miller. “You have to be so careful with communication.” 
  </p>
  <p>
    Her website and app of the same name, <a href="http://astrologyzone.com" target="_blank">Astrologyzone.com</a>, has garnered over 17 million hits amid the growing obsession with astrological phenomena and its ability to give semi-personalized insights. To the non-astrologically inclined, it is seen as merely a scapegoat for minor life inconveniences. 
  </p>
  <p>
    Mercury retrograde, according to Miller, is “the one thing that affects everybody the same way.” Although it can affect certain astrological signs differently depending on when retrograde is happening, the effects of retrograde can be felt by anyone. She <a href="https://www.astrologyzone.com/everything-you-need-to-know-about-mercury-retrograde/" target="_blank">warns</a> against signing contracts, making big life decisions, even traveling for business and booking tickets during Mercury retrograde periods. 
  </p>
  <p>
    Her advice – “slow down and be more deliberate.”
  </p>
</article>

<style>
  .container {
    border: 3px solid gray;
    max-width: 600px;
    margin: 0 auto;
    padding: 2rem 3rem 3rem;
    position: relative;
  }

  .tooltip {
    font-family: "Fira Sans", sans-serif;
    font-size: 0.8em;
    display: none;
    position: absolute;
    top: 0px;
    left: 0px;
    background: rgba(255, 255, 255, 0.9);
    color: black;
    padding: 0.3rem 0.5rem;
    width: fit-content;
    height: fit-content;
  }

  h4 {
    margin: 2rem 0 0 0;
  }

  .legend {
    margin-top: 2rem;
  }

  .legend * {
    font-family: "Fira Sans", sans-serif;
  }

  .legend text {
    font-size: 0.8em;
  }
</style>
