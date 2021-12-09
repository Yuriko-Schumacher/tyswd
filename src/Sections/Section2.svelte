<script>
  import * as d3 from 'd3';
  import Scroller from "@sveltejs/svelte-scroller";
  import { spring } from 'svelte/motion';
  import { cubicOut } from 'svelte/easing';

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

  const width = window.innerWidth / 2;
  const height = window.innerHeight;
  const margin = {
    t: 10, r: 25, b: 25, l: 25
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
          opacity: d.retrograde === "TRUE" ? 1 : 0.5,
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

<Scroller top="{0}" bottom="{1}" bind:index bind:offset bind:progress>
  <div class="scroller background" slot="background">
    <svg {width} {height}>
      <g transform="translate({margin.l}, {margin.t})">
        <g id="density-plot">
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
    <section data-section-id="3" class="step">
      <div class="step-text">
        <p>
          Section 3...
        </p>
        <p>
          Make it interactive? Or we can just remove this section.
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
  .scroller {
    width: 50%;
  }

  div.step-text {
    padding: 2rem 3rem;
    background: rgba(255, 255, 255, 0.1);
  }

  /* .background {
    background: rgba(255, 255, 255, 0.1);
  } */

  .foreground {
    /* background: rgba(255, 192, 203, 0.1); */
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
</style>