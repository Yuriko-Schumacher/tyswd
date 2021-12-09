<script>
  import * as d3 from 'd3';
  import Scroller from "@sveltejs/svelte-scroller";
  import { spring } from 'svelte/motion';

  export let data;
  let density;

  let index, offset, progress;

  data.map(d => {
    d.delay = +d.delay;
    d.delay_pct = +d.delay_pct;
    d.on_time_pct = +d.on_time_pct;
    d.total = +d.total;
  })

  let retTrueD = data.filter(d => d.retrograde === "TRUE")
  let retFalseD = data.filter(d => d.retrograde === "FALSE")
  console.log(retTrueD)
  console.log(retFalseD)

  const width = window.innerWidth * 0.5;
  const height = window.innerHeight;
  const margin = {
    t: 10, r: 50, b: 25, l: 25
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

  let xScale = d3.scaleLinear().domain([1,7]).range([margin.l, width - margin.r])
  let yScale = d3.scaleLinear().domain([0,0.8]).range([height / 2 - margin.b, margin.t])

  const vlineNumbers = [...Array(14).keys()].map(d => (d / 2 + 1));
  console.log(vlineNumbers)

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
  density =  [
    {
      retrograde: "TRUE",
      density: kde(retTrueD.map(d => d.delay_pct))
    },
    {
      retrograde: "FALSE",
      density: kde(retFalseD.map(d => d.delay_pct))
    }
  ]
  density.map(d => {
    d.line = d3.line()
      .curve(d3.curveBasis)
      .x(p => xScale(p[0]))
      .y(p => yScale(p[1]))
  })

  const move = (y) => `transform: translate(0px, ${y}px)`;

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
  
  const legendSvgs = ["normal", "ret"]
  const texts = ["Normal times", "Retrograde period"]

  $: {
    if (index === 0) {
        newPaths = density.map((d) => ({
          id: d.retrograde === "TRUE" ? 0 : 1,
          strokeWidth: 1,
          opacity: 1,
          y: 0,
          r: d.retrograde === "TRUE" ? colors.yellow.r : colors.white,
          g: d.retrograde === "TRUE" ? colors.yellow.g : colors.white,
          b: d.retrograde === "TRUE" ? colors.yellow.b : colors.white,
        }))
      } else if (index === 1) {
        newPaths = density.map((d) => ({
          id: d.retrograde === "TRUE" ? 0 : 1,
          strokeWidth: 1,
          opacity: 1,
          y: d.retrograde === "TRUE" ? height / 3 : 0,
          r: d.retrograde === "TRUE" ? colors.yellow.r : colors.white,
          g: d.retrograde === "TRUE" ? colors.yellow.g : colors.white,
          b: d.retrograde === "TRUE" ? colors.yellow.b : colors.white,
        }))
      }
      paths.set(newPaths)
  }

</script>

<section>
  <article>
    <h2>
      Mercury retrograde x flight delays
    </h2>
    <ul>
      <li>
        How Mercury gets in retrograde
        <ul>
          <li>
            Data viz: calendar?
          </li>
        </ul>
      </li>
      <li>
        General explanation and distribution of flight delays
        <ul>
          <li>
            Explain 15-minute period
          </li>
          <li>
            2020 was very unusual
          </li>
        </ul>
      </li>
      <li>
        Data viz: density plot (interactive)
        <ul>
          <li>
            Slightly delays per day when Mercury is in retrograde
          </li>
        </ul>
      </li>
    </ul>
  </article>
</section>

<Scroller top="{0}" bottom="{1}" threshold="{0.5}" bind:index bind:offset bind:progress>
  <div class="scroller background" slot="background">
    <svg {width} {height}>
      <text
        transform="translate({margin.l}, 50)"
        class="chart-title"
        fill="white">
        Mercury retrograde and flight delays
      </text>
      <g class="legend">
        {#each legendSvgs as d}
          <image
            transform="translate({d === "normal" ? margin.l : margin.l + 200}, 75)"
            width="50"
            opacity="{index === 0 ? 1 : 0}"
            href="./image/density-{d}.svg"></image>
        {/each}
      </g>
      <g class="x-axis">
        {#each vlineNumbers as n}
          <line
            transform="translate({margin.l}, {margin.t})"
            x1="{xScale(n)}"
            y1="{index === 0 ? yScale(0) : yScale(0) + height / 3}"
            x2="{xScale(n)}"
            y2="{150}"
            stroke-width="0.5"
            stroke="white"
            stroke-opacity="0.5"
          ></line> 
          <text
            class="x-axis-ticks"
            x="{xScale(n) + margin.l}"
            y="{index === 0 ? yScale(0) + 30 : yScale(0) + 30 + height / 3}"
            fill="white"
            text-anchor="middle"
          >{d3.format(",.1f")(n)}
          </text>
        {/each}
      </g>
      <g class="y-axis"></g>
      <g class="x-axis-label">
        <text
            x={width / 2 + margin.l}
            y={index === 0 ? height / 3 + height / 5 : height - margin.b - height / 10}
            text-anchor="middle"
            fill="white"
        >
          % of delayed flights, adjusted
        </text>
        <text class="axis-label"
            x={width - margin. r}
            y={index === 0 ? height / 3 + height / 5 : height - margin.b - height / 10}
            text-anchor="end"
        >
            More delays per day →
        </text>
        <text class="axis-label"
            x={margin.l + 50}
            y={index === 0 ? height / 3 + height / 5 : height - margin.b - height / 10}
            text-anchor="start"
        >
            ← Fewer delays per day
        </text>
      </g>
      <g class="y-axis-label">
        <text class="axis-label"
            transform={`translate(30, ${index === 0 ? height / 3 : height / 2}) rotate(-90)`}
            text-anchor="middle">
            Dencity
        </text>
      </g>
      <g class="x-axis-grid"></g>
      <g class="y-axis-grid"></g>
      <g transform="translate({margin.l}, {margin.t})">
        <g id="density-plot">
          {#each texts as text}
            <g class="chart-subtitle">
              <text
                transform="translate({index === 0 ? text === "Normal times" ? 55 : 255 : 50},
                                    {index === 0 ? 85 : text === "Normal times" ? 200 : height / 3 + 200})"
                fill="white"
              >
                {text}
              </text>
            </g>
          {/each}
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
  </div>

  <div class="scroller foreground" slot="foreground">
    <section data-section-id="1" class="step">
      <div class="step-text">
        <p>
          Section 1...
        </p>
        <p>
          Explain something here... Overlapped density plot
        </p>
      </div>
    </section>
    <section data-section-id="2" class="step">
      <div class="step-text">
        <p>
          Section 2...
        </p>
        <p>
          Explain something here... Top: delay % in normal days / Bottom: delay % when mercury is in retrograde
        </p>
      </div>
    </section>
  </div>
</Scroller>

<article>
  <p>
    Lorem ipsum dolor sit amet consectetur adipisicing elit. Eaque alias aspernatur animi tempora exercitationem atque. Eaque cupiditate laborum mollitia magnam, libero at architecto? Expedita optio a laboriosam ea neque iusto.
  </p>
  <p>
    Lorem ipsum dolor sit amet, consectetur adipisicing elit. Fugit expedita ipsam similique sapiente ipsum quia cumque, praesentium exercitationem nemo magni velit, recusandae ea explicabo possimus blanditiis totam, alias quod quidem.
  </p>
</article>

<style>
  div.step-text {
    padding: 2rem 3rem;
    background: rgba(255, 255, 255, 0.1);
  }

  .scroller {
    width: 50%;
  }  

  .foreground {
    margin: 0 0 0 auto;
  }

  section.step {
    width: 40vw;
    min-height: 80vh;
    margin: 0 auto;
    padding: 5rem;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
  }

  .chart-title {
    font-size: 1.5rem;
    font-weight: bold;
  }

  .axis-label {
    fill: gray;
    font-size: 0.8rem;
  }

  .x-axis-ticks {
    font-size: 0.8rem;
  }
</style>