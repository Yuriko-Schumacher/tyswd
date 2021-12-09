<script>
  import * as d3 from 'd3';
  import Scroller from "@sveltejs/svelte-scroller";
  import { spring } from 'svelte/motion';
import { onMount } from 'svelte';

  export let data;
  let index, offset, progress;

  const parseTime = d3.timeParse("%Y-%m-%d");
  const getYear = d3.timeFormat("%Y")
  data.map(d => {
    d.date = parseTime(d.date);
    d.year = +d.year;
    d.est_pct_illuminated = +d.est_pct_illuminated;
    d.pct_fixed = +d.pct_fixed;
    d.fatalities = +d.fatalities;
  })

  const width = window.innerWidth * 0.7;
  const height = window.innerHeight;
  const outerRadius = width > height ? height * 0.6 - 250 : width * 0.6 - 150 
  const margin = {
    t: 10, r: 25, b: 25, l: 25
  }

  const xScale = d3
    .scaleLinear()
    .domain([0, 200])
    .range([2 * Math.PI, 0]);

  const yScale = d3
    .scaleTime()
    .domain([parseTime("1966-01-01"), parseTime("2030-12-31")])
    .range([0, outerRadius]);

  const rScale = d3
    .scaleSqrt()
    .domain(d3.extent(data, (d) => d.fatalities))
    .range([1, 15]);

  const aScale = d3
    .scaleLinear()
    .domain(d3.extent(data, (d) => d.fatalities))
    .range([0.1, 0.8]);

  const move = (cx, cy) => `transform: translate(${cx}px, ${cy}px)`;

  const xAxisD = [...Array(4).keys()];
  const yAxisD = [
			"1970-01-01",
			"1980-01-01",
			"1990-01-01",
			"2000-01-01",
			"2010-01-01",
			"2020-01-01",
		];
  const phases = [
    {
      pct: 0,
      name:  "New Moon",
    },
    {
      pct: 25,
      name: "Waxing Crescent",
    },
    {
      pct: 50,
      name:  "First Quarter",
    },
    {
      pct: 75,
      name:  "Waxing Gibbous",
    },
    {
      pct: 100,
      name:  "Full Moon",
    },
    {
      pct: 125,
      name:  "Waning Gibbous",
    },
    {
      pct: 150,
      name:  "Third Quarter",
    },
    {
      pct: 175,
      name:  "Waning Crescent",
    },
  ]
  const fatalities = [
    {
      n: 1,
      f: 1
    },
    {
      n: 2,
      f: 50,
    },
    {
      n: 3,
      f: 100,
    },
    {
      n: 4,
      f: 150,
    },
    {
      n: 5,
      f: 200,
    }
  ]

  $: {

  }
  let imgD = [...Array(8).keys()];
	let imgSize = 65;
</script>

<section>
  <article>
    <h2>
      Moon phases x accidents
    </h2>
    <ul>
      <li>
        How the moon phases
      </li>
      <li>
        Data viz: radial scatter plot (interactive)
      </li>
      <li>
        Table: Top 30 deadliest accidents and moon phases
      </li>
    </ul>
  </article>
</section>

<div class="flourish-embed">
  <iframe
    src='https://flo.uri.sh/visualisation/8040745/embed'
    title='Interactive or visual content'
    class='flourish-embed-iframe'
    frameborder='0'
    scrolling='no'
    style='width:100%;height:500px;'
    sandbox='allow-same-origin allow-forms allow-scripts allow-downloads allow-popups allow-popups-to-escape-sandbox allow-top-navigation-by-user-activation'>
  </iframe>
  <div style='width:100%!;margin-top:4px!important;text-align:right!important;'>
    <a
      class='flourish-credit'
      href='https://public.flourish.studio/visualisation/8040745/?utm_source=embed&utm_campaign=visualisation/8040745'
      target='_top'
      style='text-decoration:none!important'>
        <img 
          alt='Made with Flourish'
          src='https://public.flourish.studio/resources/made_with_flourish.svg'
          style='width:105px!important;height:16px!important;border:none!important;margin:0!important;'> </a>
  </div>
</div>

<Scroller top="{0}" bottom="{1}" threshold="{0.5}" bind:index bind:offset bind:progress>
  <div class="scroller background" slot="background">
    <svg {width} {height}>
      <text
        transform="translate({margin.l}, 50)"
        class="chart-title"
        fill="white">
        Moon phases and fatal accidents
      </text>
      <g class="legend" transform="translate({margin.l}, 100)" opacity="{index === 2 ? 1 : 0}">
        <text
          fill="gray"
        >
        Fatalities
        </text>
        {#each fatalities as {n, f}}
          <circle
            transform="translate({n * 50 - 20}, 20)"
            r="{rScale(f)}"
            fill="yellow"
            fill-opacity="{aScale(f)}"
          ></circle>
          <text
            transform="translate({n * 50 - 20}, 50)"
            fill="gray"
            text-anchor="middle"
            class="fatalities"
          >{f}</text>
        {/each}
      </g>
        <g id="radial-scatter">
          <g transform="translate({width / 2}, {height / 2 + 50})">
            <g class="x-axis">
              {#each xAxisD as d}
                <line
                  x1="{-outerRadius / 1.4}"
                  y1="{-outerRadius / 1.4}"
                  x2="{outerRadius / 1.4}"
                  y2="{outerRadius / 1.4}"
                  transform="rotate({d * 45})"
                  stroke="white"
                  stroke-width="0.5"
                  stroke-opacity="0.3"
                ></line>
              {/each}
            </g>
            <g class="y-axis">
              {#each yAxisD as d}
                <circle
                  cx="0"
                  cy="0"
                  r="{yScale(parseTime(d))}"
                  fill-opacity="0"
                  stroke="white"
                  stroke-width="0.5"
                  stroke-opacity="0.3"
                ></circle>
              {/each}
            </g>
            <g class="x-axis-label">
              {#each phases as {pct, name}, i}
                <text
                  transform="translate(
                    {yScale(parseTime("2030-12-31")) * Math.cos(xScale(i * 25))},
                    {i <= 4 
                      ? yScale(parseTime("2030-12-31")) * Math.sin(xScale(i * 25)) - imgSize
                      : yScale(parseTime("2030-12-31")) * Math.sin(xScale(i * 25)) + imgSize})"
                  text-anchor="middle"
                  fill="white"
                >
                  {name}
                </text>
              {/each}
            </g>
            <g class="y-axis-label">
              {#each yAxisD as d}
                <text
                  transform="translate(0, {yScale(parseTime(d)) + 2})"
                  text-anchor="middle"
                  fill="white"
                  fill-opacity="{index === 0 ? 1 : 0.5}"
                >{getYear(parseTime(d))}
                </text>
              {/each}
            </g>
            <g class="dots">
              {#each data as d}
                <circle
                  style="{move(yScale(d.date) * Math.cos(xScale(d.pct_fixed)), yScale(d.date) * Math.sin(xScale(d.pct_fixed)))}"
                  r="{index <= 1 ? 1 : rScale(d.fatalities)}"
                  stroke-opacity="{index == 0 ? 0 : index == 1 ? 0.5 : aScale(d.fatalities)}"
                  fill="yellow"
                  fill-opacity="{index == 0 ? 0 : index == 1 ? 0.1 : aScale(d.fatalities)}"
                />
              {/each}
            </g>
          </g>
          <g class="imgs" transform="translate({width / 2 - imgSize / 2}, {height / 2 - imgSize / 2 + 50})">
            {#each imgD as d}
              <g>
                <image
                  class="moon-img"
                  width={imgSize}
                  height={imgSize}
                  transform="translate(
                    {yScale(parseTime("2030-12-31")) * Math.cos(xScale(d * 25))},
                    {yScale(parseTime("2030-12-31")) * Math.sin(xScale(d * 25))})"
                  href="./image/moon-phases/{d + 1}.jpeg"
                ></image>
              </g>
            {/each}
          </g>
        </g>
    </svg>
  </div>

  <div class="scroller foreground" slot="foreground">
    <section data-section-id="1" class="step">
      <div class="step-text">
        <p>
          Explanation about moon phases.
        </p>
        <p>
          It changes from New Moon -> ... -> Full Moon -> ... New Moon. It takes 29.5 days from new moon to new moon.
        </p>
        <p>
          Outer radius => closer to 2021
        </p>
      </div>
    </section>
    <section data-section-id="2" class="step">
      <div class="step-text">
        <p>
          The total number of accidents
        </p>
        <p>
          More days during new moon and full moon => not surprisingly more accidents.
        </p>
        <p>
          Also more accidents closer to the center ... as the technology increased, the number of accidents decreased.
        </p>
      </div>
    </section>
    <section data-section-id="3" class="step">
      <div class="step-text">
        <p>
          How about big accidents?
        </p>
        <p>
          Most big accidents happened ...
        </p>
      </div>
    </section>
  </div>
</Scroller>

<style>
  .flourish-embed {
    margin: 0 auto;
    width: 80%;
    max-width: 1000px;
  }

  section.step {
    max-width: 600px;
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
  }

  div.step-text {
    padding: 2rem 3rem;
    background: rgba(255, 255, 255, 0.1);
  }

  .background {
    width: 70vw;
  }

  .foreground {
    width: 30vw;
    margin: 0 0 0 auto;
  }

  section.step {
    width: 30vw;
    min-height: 80vh;
    margin: 0 auto 0 0;
    padding: 0 5rem 0 0;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
  }

  .chart-title {
    font-size: 1.5rem;
    font-weight: bold;
  }

  .fatalities {
    font-size: 0.8rem;
  }

  .x-axis-label text {
    font-size: 0.8rem;
  }

  .y-axis-label text {
    font-size: 0.8rem;
  }
</style>
