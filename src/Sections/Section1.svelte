<script>
  import * as d3 from 'd3';
  import Scroller from "@sveltejs/svelte-scroller";
  import { spring } from 'svelte/motion';

  export let data;
  let index, offset, progress;

  const parseTime = d3.timeParse("%Y-%m-%d");
  data.map(d => {
    d.date = parseTime(d.date);
    d.year = +d.year;
    d.est_pct_illuminated = +d.est_pct_illuminated;
    d.pct_fixed = +d.pct_fixed;
    d.fatalities = +d.fatalities;
  })
  console.log(data)

  const width = window.innerWidth * 0.6;
  const height = window.innerHeight;
  const outerRadius = width > height ? height * 0.6 - 150 : width * 0.6 - 150 
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

  console.log(rScale(1))


  const move = (cx, cy) => `transform: translate(${cx}px, ${cy}px)`;
  const colors = {
    yellow : {
      r: 255,
      g: 255,
      b: 0
    },
    white : 238,
    black : 34
  }
  let dots = spring(
    data.map((d) => ({
      cx: yScale(d.date) * Math.cos(xScale(d.pct_fixed)),
      cy: -yScale(d.date) * Math.sin(xScale(d.pct_fixed)),
      cr: rScale(d.fatalities),
      strokeWidth: 0.4,
      opacity: aScale(d.fatalities),
      r: colors.yellow.r,
      g: colors.yellow.g,
      b: colors.yellow.b,
    })),
    {
      stiffness: 0.1,
      damping: 0.9
    }
  );
  let newDots;

  // $: {
  //   if (index === 0) {
  //     newDots = data.map((d) => ({
  //       cx: yScale(d.date) * Math.cos(xScale(d.pct_fixed)),
  //       cy: -yScale(d.date) * Math.sin(xScale(d.pct_fixed)),
  //       cr: 1,
  //       strokeWidth: 0.4,
  //       opacity: aScale(d.fatalities),
  //       r: colors.yellow.r,
  //       g: colors.yellow.g,
  //       b: colors.yellow.b,
  //     }))
  //   }
  //   dots.set(newDots)
  // }
  let imgD = [...Array(8).keys()];
	let imgSize = 75;
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

<Scroller top="{0}" bottom="{1}" bind:index bind:offset bind:progress>
  <div class="scroller background" slot="background">
    <svg {width} {height}>
        <g id="radial-scatter">
          <g transform="translate({width / 2}, {height / 2})">
            <g class="dots">
              {#each $dots as { strokeWidth, opacity, cx, cy, cr, r, g, b}}
                <circle
                  style="{move(cx, cy)}"
                  r="{cr}"
                  stroke="#cccccc"
                  stroke-width="{strokeWidth}"
                  stroke-opacity="{opacity}"
                  fill="rgb({r}, {g}, {b})"
                  fill-opacity="{opacity}"
                />
              {/each}
            </g>
          </g>
          <g class="imgs" transform="translate({width / 2 - imgSize / 2}, {height / 2 - imgSize / 2})">
            {#each imgD as d}
              <g>
                <image
                  width={imgSize}
                  height={imgSize}
                  transform="translate(
                    {yScale(parseTime("2030-12-31")) * Math.cos(xScale(d * 25))},
                    {-yScale(parseTime("2030-12-31")) * Math.sin(xScale(d * 25))})"
                  href="./image/moon-phases/{d + 1}.jpg"
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
          Section 1...
        </p>
        <p>
          Some explanation... Moon phases x fatal accidents radial scatter plot. Circle sizes = fatalities. Big accidents happened when...
        </p>
      </div>
    </section>
    <section data-section-id="2" class="step">
      <div class="step-text">
        <p>
          Section 2...
        </p>
        <p>
          Some more explanation... Look for some data point to highlight? 9/11 attack?
        </p>
      </div>
    </section>
    <section data-section-id="3" class="step">
      <div class="step-text">
        <p>
          Section 3...
        </p>
        <p>
          Some explanation... Make it interactive? Or we can just remove this section.
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
    width: 50%;
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

  .scroller {
    width: 50%;
  }

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
