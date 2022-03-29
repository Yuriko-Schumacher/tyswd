<script>
  import * as d3 from 'd3';
  import Scroller from "@sveltejs/svelte-scroller";
  import { spring } from 'svelte/motion';

  export let data;
  export let width;
  export let height;

  let w, h, xScale, yScale, vlineNumbers;
  let index, offset, progress;

  data.map(d => {
    d.delay = +d.delay;
    d.delay_pct = +d.delay_pct;
    d.on_time_pct = +d.on_time_pct;
    d.total = +d.total;
  })

  let retTrueD = data.filter(d => d.retrograde === "TRUE")
  let retFalseD = data.filter(d => d.retrograde === "FALSE")

  const margin = width > 467 ? {
    t: 10, r: 120, b: 25, l: 50
  } : {
    t: -100, r: 50, b: 25, l: 20
  }
  const colors = {
    yellow : {
      r: 255,
      g: 255,
      b: 0
    },
    white : 238,
    black : 34
  }
  const legendSvgs = [
    { type: "normal", text: "Normal times"},
    { type: "ret", text: "Retrograde period" }
  ]
  const texts = ["Normal times", "Retrograde period"]
  const move = (y) => `transform: translate(0px, ${y}px)`;

  let density =  [{ retrograde: "TRUE" }, { retrograde: "FALSE" }]
  let paths = spring(
    density.map((d) => ({
      id: d.retrograde === "TRUE" ? 0 : 1,
      strokeWidth: 1,
      opacity: 1,
      y: 0,
      r: d.retrograde === "TRUE" ? colors.yellow.r : colors.white,
      g: d.retrograde === "TRUE" ? colors.yellow.g : colors.white,
      b: d.retrograde === "TRUE" ? colors.yellow.b : colors.white,
    })),
    {
      stiffness: 0.1,
      damping: 0.9
    }
  );
  let newPaths;

  $: {
    if (width !== undefined && height !== undefined) {
      w = width > 1000 ? width * 0.5: width;
      h = height * 0.9;
      vlineNumbers = [...Array(13).keys()].map(d => (d / 2 + 1))
      xScale = d3.scaleLinear().domain([1,7]).range([margin.l, w - margin.r])
      yScale = d3.scaleLinear().domain([0,0.8]).range([h / 2 - margin.b, margin.t])

      // Function to compute density
      const kernelDensityEstimator = (kernel, X) => {
        return function(V) {
          return X.map(function(x) {
            return [x, d3.mean(V, function(v) { return kernel(x - v); })];
          });
        };
      }
      const kernelEpanechnikov = (k) => {
        return function(v) {
          return Math.abs(v /= k) <= 1 ? 0.75 * (1 - v * v) / k : 0;
        };
      }
      // Compute kernel density estimation
      const kde = kernelDensityEstimator(kernelEpanechnikov(0.1), xScale.ticks(20))
      density.map(d => {
        d.density = 
          d.retrograde == "TRUE" ? kde(retTrueD.map(d => d.delay_pct)) : kde(retFalseD.map(d => d.delay_pct))
        d.line = d3.line()
          .curve(d3.curveBasis)
          .x(p => xScale(p[0]))
          .y(p => yScale(p[1]))
      })

      if (index <= 1) {
        newPaths = density.map((d) => ({
          id: d.retrograde === "TRUE" ? 0 : 1,
          strokeWidth: 1,
          opacity: 1,
          y: 0,
          r: d.retrograde === "TRUE" ? colors.yellow.r : colors.white,
          g: d.retrograde === "TRUE" ? colors.yellow.g : colors.white,
          b: d.retrograde === "TRUE" ? colors.yellow.b : colors.white,
        }))
      } else {
        newPaths = density.map((d) => ({
          id: d.retrograde === "TRUE" ? 0 : 1,
          strokeWidth: 1,
          opacity: 1,
          y: d.retrograde === "TRUE" ? h / 3 : 0,
          r: d.retrograde === "TRUE" ? colors.yellow.r : colors.white,
          g: d.retrograde === "TRUE" ? colors.yellow.g : colors.white,
          b: d.retrograde === "TRUE" ? colors.yellow.b : colors.white,
        }))
      }
      paths.set(newPaths)
    }
  }

</script>

<Scroller top="{0}" bottom="{1}" threshold="{0.5}" bind:index bind:offset bind:progress>
  <div class="scroller background" slot="background">
    {#if width !== undefined && height !== undefined}
      <div class="chart-title">
        <h2>Mercury retrograde and flight delays</h2>
        <div class="legend">
          {#each legendSvgs as d}
            <div class="legend-item">
              <img
                width="50"
                opacity="{index <= 1 ? 1 : 0}"
                src="./image/density-{d.type}.svg"
                alt="density plot legends"/>
              {d.text}
            </div>
          {/each}
        </div>
      </div>
      <svg width={w} height={h}>
        <g class="x-axis">
          {#each vlineNumbers as n}
            <line
              transform="translate({margin.l}, {margin.t})"
              x1="{xScale(n)}"
              y1="{index <= 1 ? yScale(0) : yScale(0) + h / 3}"
              x2="{xScale(n)}"
              y2="{width > 467 ? 150 : 60}"
              stroke-width="0.3"
              stroke="white"
              stroke-opacity="0.5"
            ></line> 
            {#if width > 467}
              <text
                class="x-axis-ticks"
                x="{xScale(n) + margin.l}"
                y="{index <= 1 ? yScale(0) + 30 : yScale(0) + 30 + h / 3}"
                fill="white"
                text-anchor="middle"
              >{d3.format(",.1f")(n)}
              </text>
            {:else}
              <text
                class="x-axis-ticks"
                x="{xScale(n) + margin.l}"
                y="{index <= 1 ? yScale(0) - 80 : yScale(0) - 80 + h / 3}"
                fill="white"
                text-anchor="middle"
              >{d3.format(",.1f")(n)}
              </text>
            {/if}
          {/each}
        </g>
        <g class="y-axis"></g>
        {#if width > 467}
          <g class="x-axis-label">
            <text
                x={w / 2 + margin.l}
                y={index <= 1 ? h / 3 + h / 5 + 30 : h - margin.b - h / 10 + 30}
                text-anchor="middle"
                fill="gray"
            >
              % of delayed flights, adjusted
            </text>
            <text class="axis-label"
                x={w - margin.r / 2}
                y={index <= 1 ? h / 3 + h / 5 : h - margin.b - h / 10}
                text-anchor="end"
            >
                More delays per day →
            </text>
            <text class="axis-label"
                x={margin.l + 50}
                y={index <= 1 ? h / 3 + h / 5 : h - margin.b - h / 10}
                text-anchor="start"
            >
                ← Fewer delays per day
            </text>
          </g>
        {:else}
          <g class="x-axis-label">
            <text
                x={w / 2 + margin.l}
                y={index <= 1 ? h / 3 + h / 5 - 110 : h - margin.b - h / 10 - 110}
                text-anchor="middle"
                fill="gray"
            >
              % of delayed flights, adjusted
            </text>
            <text class="axis-label"
                x={w - margin.r / 2}
                y={index <= 1 ? h / 3 + h / 5 - 90 : h - margin.b - h / 10 - 90}
                text-anchor="end"
            >
                More delays per day →
            </text>
            <text class="axis-label"
                x={margin.l + 30}
                y={index <= 1 ? h / 3 + h / 5 - 90 : h - margin.b - h / 10 - 90}
                text-anchor="start"
            >
                ← Fewer delays per day
            </text>
          </g>
        {/if}
        <g class="y-axis-label">
          <text
              transform={`translate(${width > 467 ? 80 : 30}, ${index <= 1 ? h / 3 - 60 : height / 2 - 60}) rotate(-90)`}
              text-anchor="middle"
              fill="gray">
              Density
          </text>
        </g>
        <g class="x-axis-grid"></g>
        <g class="y-axis-grid"></g>
        <g transform="translate({margin.l}, {margin.t})">
          <g id="density-plot">
            {#if width > 467}
              {#each texts as text}
                <g class="chart-subtitle">
                  <text
                    transform="translate({index <= 1 ? text === "Normal times" ? 55 : 255 : 50},
                    {index <= 1 ? 85 : text === "Normal times" ? 200 : h / 3 + 200})"
                    fill="white"
                    opacity={index <= 1 ? 0 : 1}
                  >
                    {text}
                  </text>
                </g>
              {/each}
            {/if}
            {#each $paths as { id, strokeWidth, opacity, y, r, g, b}}
              <path
                style="{move(y)}; opacity: {opacity}"
                data-id="{density[id].density}"
                d="{density[id].line(density[id].density)}"
                fill="rgb({r}, {g}, {b})"
                fill-opacity="0.4"
                stroke-width="{strokeWidth}"
                stroke="rgb({r}, {g}, {b})"
              ></path>
            {/each}
          </g>
        </g>
      </svg>
    {/if}
  </div>

  <div class="scroller foreground" slot="foreground">
    <section data-section-id="0" class="step">
    </section>
    <section data-section-id="1" class="step">
      <div class="step-text">
        <p>
          For two thirds of the year, while Mercury is not in retrograde, 3.5% to 4.0% of flights are delayed on a daily basis. 
        </p>
        <p>
          The spike in delays related to the COVID-19 pandemic in late March 2020 did not occur during retrograde, but the bump around 5.25% is not linked to any specific time period. 
        </p>
      </div>
    </section>
    <section data-section-id="2" class="step">
      <div class="step-text">
        <p>
          During Mercury retrograde, however, the two peaks shift slightly to approximately 3.7% and 4.25%. Until the 5% mark, the percentage of flights delayed during retrograde remains higher than that during the normal period — suggesting there may be an undetectable increase in delays.
        </p>
      </div>
    </section>
    <section data-section-id="3" class="step">
    </section>
  </div>
</Scroller>

  <article>
    <p>
      Sriwijaya Air Flight 182 did not occur during the earliest retrograde period this year, which lasted from Jan. 30 to Feb. 20. It was, however, delayed due to bad weather.
    </p>
    <p>
      Mercury can enter its retrograde cycle during one month or star sign and exit in another. Within the 37 times in the past 11 years, retrograde began 20 times in the six colder months of the year – October through March – and 17 times during the warmer half from April through September. 
    </p>
    <p>
      According to the <a href="https://www.faa.gov/newsroom/inclement-weather-0?newsId=23074" target="_blank">Federal Aviation Administration</a>, inclement weather creates potentially hazardous conditions for flights. In an average year, bad weather accounts for nearly 70% of flight delays. 
    </p>
  </article>

<style>
  div.step-text {
    padding: 2rem 3rem;
    background: rgba(0, 0, 30, 0.85);
  }
  .scroller {
    width: 50%;
  }  
  .foreground {
    width: 50vw;
    margin: 0 0 0 auto;
  }
  .step {
    width: 50vw;
    max-width: 600px;
    min-height: 80vh;
    margin: 30vh auto 30vh 10%;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
  }
  .step:last-of-type {
    margin-bottom: 0;
  }
  .chart-title {
    width: calc(100% - 50px);
    padding: 3rem 0 0 0;
    margin: 0 auto;
  }
  .chart-title h2 {
    font-size: 1.5rem;
  }
  .legend {
    display: flex;
    flex-direction: row;
    gap: 1em;
  }
  .axis-label {
    fill: gray;
    font-size: 0.8rem;
  }
  .x-axis-ticks {
    font-size: 0.8rem;
  }
  @media (max-width: 1000px) {
    .scroller {
      width: 100%;
    }
    .foreground {
      width: 100vw;
    }
    .step {
      width: 100vw;
      margin: 50vh auto;
      padding: 0;
    }
  }
  @media (max-width: 467px) {
    .chart-title {
      padding-top: 1rem;
    }
    .legend {
      flex-direction: column;
      gap: 0.2em;
    }
  }
</style>