<script>
  import * as d3 from 'd3';

  export let data;
  let index, offset, progress;

  data.map(d => {
    d.fatalities = +d.fatalities;
    d.total_accidents = +d.total_accidents;
  })

  data = data.filter(d => d.fatalities <= 7)

  const width = window.innerWidth / 2
  const height = width * 0.5;
  const margin = {
    t: 100, r: 50, b: 50, l: 50
  }

  const group = [...Array(7).keys()].map(d => d + 1)
  const subGroup = ["TRUE", "FALSE"]

  const xScale = d3.scaleBand()
    .domain(group)
    .range([margin.l, width - margin.r - margin.l])
    .padding([0.2])

  const yScale = d3.scaleLinear()
    .domain([0, 350])
    .range([height - margin.b, margin.t])

  const xSubScale = d3.scaleBand()
    .domain(subGroup)
    .range([0, xScale.bandwidth()])
    .padding([0.2])

  const hlines = [...Array(7).keys()].map(d => d * 50)
</script>

<article>
  <h2>
    On the importance of threes
  </h2>
  <p>
    Mercury represents the “Third House” in astrology – the house of communication – and moves through three stages of retrograde for three weeks every three months. If statistically significant, it would be uncanny to see an increase in the number of crashes with fatalities of exactly three people. Most smaller planes, though, can barely seat up to four people. 
  </p>
</article>

<div class="grouped-bar-chart">
  <div class="title">
    <h3>
      Number of fatalities in individual crashes are even between normal and retrograde periods, except when there are three fatalities.
    </h3>
  </div>
  <svg width={width} height={height}>
    <g class="legend">
      {#each subGroup as {g}, i}
        <rect
          x="{margin.l + i * 200}"
          y="50"
          width="{xSubScale.bandwidth()}"
          height="{xSubScale.bandwidth()}"
          fill="{i === 0 ? "yellow" : "gray"}"
        ></rect> 
        <text
          x="{margin.l + xSubScale.bandwidth() * 1.5 + i * 200}"
          y="{50 + xSubScale.bandwidth()}"
          fill="white"
        >{i === 0 ? "Retrograde period" : "Normal times"}</text>
      {/each}
    </g>      
    {#each hlines as n}
      <line
        x1="{xScale(1)}"
        x2="{xScale(7) + xSubScale.bandwidth() * 2}"
        y1="{yScale(n)}"
        y2="{yScale(n)}"
        stroke-width="0.5"
        stroke="gray"
      ></line>
      <text
        class="y-axis-ticks"
        x="{margin.l}"
        y="{yScale(n)}"
        fill="white"
        text-anchor="middle"
      >{n}
      </text>
    {/each}
    {#each group as g}
      <text
        class="x-axis-ticks"
        x="{xScale(g) + xSubScale.bandwidth() * 1.5}"
        y="{yScale(0) + 20}"
        fill="white"
        text-anchor="middle"
      >{g}
      </text>
    {/each}
    <g class="x-axis-label">
      <text
          x={width / 2}
          y={height - margin.b + 50}
          text-anchor="end"
          fill="gray"
      >
        Fatalities
      </text>
    </g>
    <g class="y-axis-label">
      <text
          transform={`translate(15, ${height / 2 + 50}) rotate(-90)`}
          text-anchor="middle"
          fill="gray">
          Total number of accidents
      </text>
    </g>
    {#each data as d}
      <g transform="translate({xScale(d.fatalities)}, 0)">
        <rect
          x="{xSubScale(d.retrograde)}"
          y="{yScale(d.total_accidents)}"
          width="{xSubScale.bandwidth()}"
          height="{height - margin.b - yScale(d.total_accidents)}"
          fill="{d.retrograde === "TRUE" ? "yellow" : "gray"}"
          fill-opacity="{d.retrograde === "TRUE" ? 1 : 0.8}"
        ></rect>
      </g>
    {/each}
  </svg>
  <div class="note">
    <p>
      Note: There have been 840 fatal accidents during the retrograde period. Data for normal times were randomely sampled. When Mercury was in retrograde, there were one accident that killed eight people, and another that killed 10. The sample data of normal times had one accident with nine fatatlities.
    </p>
  </div>
</div>

<article>
  <p>
    “Three is a really big number with mercury,” added Campagna, who also mentioned the significance of “the third road. Mercury is the psychopomp that guides souls who have recently departed through the underworld, so they can make that transition.” 
  </p>
  <p>
    The expanding field of astrology remains largely debated – however, in a world filled with uncertainties and fears of the future, many look for answers or explanations to personal life events. Astrology prevails as one of many outlets.
  </p>
  <p>
    While there may be strange correlations between aviation and astrology, intersections of celestial realms with topics as grave as flight crashes remain largely unstudied. Pilot Smith, who has spent over 30 years flying, commented: 
  </p>
  <p>
    “Everyone has their own personal superstitions, to some degree, but overall I can’t think of a less superstitious place than a jetliner cockpit.” 
  </p>
</article>


<style>
  .grouped-bar-chart {
    border: 3px solid gray;
    width: 50vw;
    margin: 0 auto;
    padding: 2rem 3rem 3rem;
    position: relative;
  }

  .title {
    width: 80%;
    margin: 0 auto;
  }

  .note {
    width: 80%;
    margin: 0 auto;
    font-size: 0.8rem;
    font-style: italic;
    color: lightgray;
  }

  .x-axis-ticks, .y-axis-ticks {
    font-size: 0.8rem;
  }

  article {
    margin-bottom: 10rem;
  }
</style>