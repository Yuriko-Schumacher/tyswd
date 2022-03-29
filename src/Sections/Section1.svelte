<script>
  import * as d3 from 'd3';
  import Scroller from "@sveltejs/svelte-scroller";
  import { afterUpdate, onMount } from 'svelte';

  export let data;
  export let width;
  export let height;

  let w, outerRadius, imgSize, xScale, yScale, rScale, aScale;
  let index, offset, progress;

  const parseTime = d3.timeParse("%Y-%m-%d");
  const formatTime = d3.timeFormat("%B %d, %Y");
  const getYear = d3.timeFormat("%Y")
  const getMoonPhase = (pct) => {
    let roundPct = (Math.round(pct * 0.04) / 0.04)
    let phase = roundPct === 200 ? "New Moon" :
      phases.filter(d => d.pct === roundPct)[0].name
    return phase
  }

  data.map(d => {
    d.date = parseTime(d.date);
    d.year = +d.year;
    d.est_pct_illuminated = +d.est_pct_illuminated;
    d.pct_fixed = +d.pct_fixed;
    d.fatalities = +d.fatalities;
  })
  data.sort((a, b) => (a.fatalities > b.fatalities))

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
      name: "New Moon",
      side: "right",
    },
    {
      pct: 25,
      name: "Waxing Crescent",
      side: "right",
    },
    {
      pct: 50,
      name: "First Quarter",
      side: "middle",
    },
    {
      pct: 75,
      name: "Waxing Gibbous",
      side: "left",
    },
    {
      pct: 100,
      name: "Full Moon",
      side: "left",
    },
    {
      pct: 125,
      name: "Waning Gibbous",
      side: "left"
    },
    {
      pct: 150,
      name: "Third Quarter",
      side: "middle",
    },
    {
      pct: 175,
      name: "Waning Crescent",
      side: "right",
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
  const imgD = [...Array(8).keys()];
  const margin = {
    t: 10, r: 25, b: 25, l: 25
  }

  $: {
    w = width > 1000
      ? width * 0.66
      : width;
    outerRadius = width > 467
      ? width > height 
        ? height * 0.5 - 250
        : w * 0.5 - 150
      : width > height
        ? height - 250
        : w - 230;
    imgSize = outerRadius / 6;

    xScale = d3
    .scaleLinear()
    .domain([0, 200])
    .range([2 * Math.PI, 0]);

    yScale = d3
      .scaleTime()
      .domain([parseTime("1966-01-01"), parseTime("2030-12-31")])
      .range([0, outerRadius]);

    rScale = d3
      .scaleSqrt()
      .domain(d3.extent(data, (d) => d.fatalities))
      .range([1, outerRadius / 25]);

    aScale = d3
      .scaleLinear()
      .domain(d3.extent(data, (d) => d.fatalities))
      .range([0.1, 0.8]);

    if (width > 467) {
      afterUpdate(() => {
        // ----- TOOLTIP -----
        const backgroundContainer = d3.select(".scroller__radial-scatter").select(".background-container")
        const circles = backgroundContainer.select(".dots").selectAll("circle")
        const tooltip = backgroundContainer.select(".tooltip")

        if (index === 8) {
          backgroundContainer.style("z-index", "0").style("pointer-events", "none")
        }

        if (index === 9) {
          backgroundContainer.style("z-index", "999").style("pointer-events", "auto")
          circles.on("mouseover", function(e) {
          let id = d3.select(this).attr("id");
          let thisD = data.filter(d => d.accident_num == id)[0]
          let cx = +d3.select(this).attr("cx") + w / 2
          let cy = +d3.select(this).attr("cy") + height / 2 + 50
          d3.select(this).attr("fill", "yellow").attr('fill-opacity', 1).attr("stroke-opacity", 1)
          tooltip
            .style("display", "block")
            .style("top", `${cy + 5}px`)
            .style("left", `${cx + 5}px`)
            .html(`Fatalities: <strong>${thisD.fatalities}</strong><br>
                    Moon phase: <strong>${getMoonPhase(thisD.pct_fixed)}</strong><br>
                    Moon illumination: <strong>${thisD.est_pct_illuminated}</strong>%<br>
                    ${formatTime(thisD.date)}<br>
                    Location: ${thisD.city}, ${thisD.state}`)
          })
          circles.on("mouseout", function() {
            let id = d3.select(this).attr("id");
            let thisD = data.filter(d => d.accident_num == id)[0]
            d3.select(this).attr("fill", "white").attr('fill-opacity', aScale(thisD.fatalities)).attr("stroke-opacity", aScale(thisD.fatalities))
            tooltip
              .style("display", "none")
          })
        }
      })
    }
  }
</script>

{#if w !== undefined}
  <div class="scroller__radial-scatter">
    <Scroller top="{0}" bottom="{1}" threshold="{0.3}" bind:index bind:offset bind:progress>
      <div class="scroller background" slot="background">
        <div class="tooltip"></div>
        <div class="chart-title">
          Moon phases and fatal accidents
        </div>
        <svg width="{w}" height="{height}">
          <g
            class="legend"
            transform="translate({margin.l}, 25)"
            opacity="{index >= 2 ? 1 : 0}">
            <text
              fill="gray"
            >
              Fatalities
            </text>
            {#each fatalities as {n, f}}
              <circle
                transform="translate({n * 50 - 20}, 20)"
                r="{rScale(f)}"
                fill="white"
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
            <g transform="translate({w / 2}, {height / 2})">
              <g class="x-axis">
                {#each xAxisD as d}
                  <line
                    x1="{-outerRadius / 1.4}"
                    y1="{-outerRadius / 1.4}"
                    x2="{outerRadius / 1.4}"
                    y2="{outerRadius / 1.4}"
                    transform="rotate({d * 45})"
                    stroke="white"
                    stroke-w="0.5"
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
                    stroke-w="0.5"
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
                    cx="{yScale(d.date) * Math.cos(xScale(d.pct_fixed))}"
                    cy="{yScale(d.date) * Math.sin(xScale(d.pct_fixed))}"
                    r="{index <= 2 ? 1 : rScale(d.fatalities)}"
                    stroke-opacity="{index == 0 ? 0
                      : index == 2 ? 0.1
                      : index >= 3 ? aScale(d.fatalities)
                      : index == 4 ? d.fatalities >= 200 ? 1 : aScale(d.fatalities)
                      : index == 5 ? d.fatalities >= 200 && d.pct_fixed < 25 ? 1 : aScale(d.fatalities)
                      : index == 6 ? d.fatalities >= 200 && d.pct_fixed > 25 ? 1 : aScale(d.fatalities)
                      : index == 7 ? d.pct_fixed >= 150 || d.pct_fixed < 25 ? aScale(d.fatalities) : aScale(d.fatalities)
                      : aScale(d.fatalities)}"
                    fill="{index <= 3 ? "white"
                      : index == 4 ? d.fatalities >= 200 ? "yellow" : "white"
                      : index == 5 ? d.fatalities >= 200 && d.pct_fixed < 25 ? "yellow" : "white"
                      : index == 6 ? d.fatalities >= 200 && d.pct_fixed > 25 ? "yellow" : "white"
                      : index == 7 ? d.pct_fixed >= 150 || d.pct_fixed < 25 ? "yellow" : "white"
                      : index == 8 && w <= 467
                        ? d.pct_fixed >= 150 || d.pct_fixed < 25
                        ? "yellow" : "white"
                      : "white"}"
                    fill-opacity="{index == 0 ? 0
                      : index == 2 ? 0.1
                      : index == 3 ? aScale(d.fatalities)
                      : index == 4 ? d.fatalities >= 200 ? 1 : aScale(d.fatalities)
                      : index == 5 ? d.fatalities >= 200 && d.pct_fixed < 25 ? 1 : aScale(d.fatalities)
                      : index == 6 ? d.fatalities >= 200 && d.pct_fixed > 25 ? 1 : aScale(d.fatalities)
                      : aScale(d.fatalities)}"
                    id="{d.accident_num}"
                  />
                {/each}
              </g>
            </g>
            <g class="imgs" transform="translate({w / 2 - imgSize / 2}, {height / 2 - imgSize / 2})">
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
        <section data-section-id="0" class="step">
        </section>
        <section data-section-id="1" class="step">
          <div class="step-text">
            <p>
              The moon itself does not generate light, as it is lit up by the sun. As the moon orbits Earth and revolves counterclockwise, the portion of illuminated moon visible from Earth changes daily. 
            </p>
            <p>
              It takes 29.5 days, roughly one month, for the moon to expand in illumination from new to full (waxing) and back to new (waning) again. 
            </p>
            <p>
              In this plot, the outer radius represents the decades closer to 2021. 
            </p>
          </div>
        </section>
        <section data-section-id="2" class="step">
          <div class="step-text">
            <p>
              Between November 1966 and 2021, there were 24,714 private and commercial flight crashes within the United States and its territories, according to <a href="https://data.ntsb.gov/carol-main-public/landing-page" target="_blank">NTSB</a>. 
            </p>
            <p>
              Although the moon spends roughly the same amount of time in each phase, there were notably more crashes closer to the New and Full moon, when the moon’s illumination creeps toward 0 and 100% respectively. 
            </p>
            <p>
              Luckily, airplane safety features have been enhanced over the last 50 years. Fewer crashes are seen happening after the 2000s, partially due to technological developments. 
            </p>
          </div>
        </section>
        <section data-section-id="3" class="step">
          <div class="step-text">
            <p>
              But when the number of fatalities is taken into consideration, the chart tells a different story.
            </p>
            <p>
              The sizes and opacities of the circles are computed based on the number of people killed.
            </p>
          </div>
        </section>
        <section data-section-id="4" class="step">
          <div class="step-text">
            <p>
              The most lethal crashes, four with more than 200 fatalities, occurred between the ‘70s and early 2000s. 
            </p>
          </div>
        </section>
        <section data-section-id="5" class="step">
          <div class="step-text">
            <p>
              Two of these four occurred <span>between the new and earliest signs of waxing crescent</span>, when the moon grows from 0% to 25% illumination.
            </p>
          </div>
        </section>
        <section data-section-id="6" class="step">
          <div class="step-text">
            <p>
              The other two occurred in close proximity to the new moon, but <span>during the final transition from waning crescent back to its origin</span>.
            </p>
          </div>
        </section>
        <section data-section-id="7" class="step">
          <div class="step-text">
            <p>
              More deadly crashes occered <span>between the third quarter and warning crescent</span>.
            </p>
            <p>
              The night sky is darker during these time periods in comparison to the light provided by the full moon – but most crashes occurred during the daytime. 
            </p>
          </div>
        </section>
        <section data-section-id="8" class="step">
          <div class="step-text">
            <p>
              Hover over the circles and explore plain crashes and the moon phases.
            </p>
            <p class="arrow">
              &darr;
            </p>
          </div>
        </section>
        <section data-section-id="9" class="step">
        </section>
      </div>
    </Scroller>
  </div>
{/if}

<article>
  <p>
    While the incidence of flight crashes has steadily decreased throughout the decades, Campagna reasoned the new moon “can be a time of more accidents because people are more tired, they’re more sad. They have less resources to function well, or emotionally they just might be feeling a little bit darker and depleted. That can contribute to how people are present.” 
  </p>
  <p>
    “Almost every emergency, unsafe situation or serious malfunction is addressed through a step-by-step sequence,” said Patrick Smith, a first officer at Delta Airlines and current pilot of Boeing 757 and 767s. 
  </p>
  <p>
    Smith, author of “The Cockpit Confidential” and an extensive airline blog, mentioned “the worlds of private aviation and commercial airline operations are vastly different realms. Safety-wise, they differ across the board, so much so that it’s difficult to compare them.” 
  </p>
  <p>
    In an interview with a current corporate pilot who decided to remain anonymous due to security reasons, he explained “standardization with the flight crews is one of the most important things regardless of the size of the plane.”
  </p>
  <p>
    “When you fly with someone you don’t know, you don’t know if they’re equipped to handle an emergency situation the same way as someone you’ve worked with before,” said the pilot with nearly 50 years of flying experience. You’d hope they follow the same steps as you do.” 
  </p>
  <p>
    For him, the biggest safety concern is not the size of the plane nor the weather. It’s automation and the growing dependency pilots have on technology to safely guide the plane.
  </p>
  <p>
    “In the old days, we flew and we had what was called ‘stick and rudder’ skills,” explained the pilot, who used to fly the largely discontinued Boeing 727s. “Planes crash a lot of the time because of errors in the cockpit. People don’t understand the systems as good as they should and this goes back, you know. It’s not just happening in the year 2021, this goes back a long time – 40, 50 years. People crash because they’re not paying attention and today, the pilots are button pushers and computer managers.” 
  </p>
  <p>
    His rule of thumb is to “aviate, navigate, and then communicate. Fly the airplane, keep it going where you're supposed to be going. And then call the controller and tell them you have a problem.”
  </p>
  <p>
    Astrologers believe a separate heavenly body orbiting Earth is in charge of communication and technology.
  </p>
  <p>
    Campagna uses the transcentury “American Ephemeris,” a guidebook that tracks navigation of celestial bodies for a hundred years starting in 1950, to accompany clients’ individual readings. But for astrologers like her, there is much more to stargazing than zodiacs and horoscope charts – their study is rooted in reading and deciphering tables of mathematical calculations of planetary trajectories. 
  </p>
  <h1>Mercury retrograde</h1>
  <p>
    For three weeks roughly every three months, mercury’s orbit appears to position itself in an opposite direction to that of surrounding planets. As the planet closest to the sun and also the smallest, Mercury embodies the worlds of communication, transportation and technology in astrological circles.
  </p>
  <p>
    Rosenfield elaborated that Mercury’s placement in one’s birth chart can influence how one communicates with others and with themselves. 
  </p>
  <p>
    Mercury is believed to have “dominion over a huge, huge aspect of our lives, because communication has really exploded over the last 40 years with the advent of the Internet and advanced technology,” added Campagna. 
  </p>
  <p>
    For two thirds of the year, Mercury’s direct motion provides the baseline expectation for awareness and analysis. During the other third, however, the planet appears to move backward and meet with the sun, signifying the state in which communication is altered.
  </p>
  <p>
    The small planet entered apparent retrograde motion 37 times from the beginning of 2010 through 2021, amounting to 840 days total. When an equally sized population of delayed flight arrivals lasting over 15 minutes were sampled to pair with non-retrograde dates, the results of Mercury’s potential impacts were somewhat unclear.
  </p>
</article>

<style>
  h1 {
    margin-top: 2.5em;
  }
  div.step-text {
    padding: 2rem 3rem;
    background: rgba(0, 0, 30, 0.85);
  }
  .background {
    width: 66vw;
  }
  .foreground {
    width: 33vw;
    margin: 0 0 0 auto;
  }
  .step {
    width: 33vw;
    max-width: 600px;
    min-height: 80vh;
    margin: 30vh auto 30vh 0;
    padding: 0 5rem 0 0;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
  }
  .step:last-of-type {
    margin: 0 auto 0 0;
  }
  span {
    background: rgba(255, 255, 0, 0.5);
    padding: 0 0.3em;
    border-radius: 0.1rem;
  }
  .arrow {
    font-size: 2rem;
    text-align: center;
  }
  .chart-title {
    width: calc(100% - 50px);
    padding: 3rem 0 0 0;
    margin: 0 auto;
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
  .tooltip {
    display: none;
    position: fixed;
    top: 0;
    left: 0;
    background: rgba(255, 255, 255, 0.9);
    color: black;
    padding: 0.8em 1em;
    border-radius: 0.1rem;
    z-index: 1000;
    font-family: "Fira Sans", sans-serif;
    font-size: 0.8em;
  }
  @media (max-width: 1000px) {
    .background {
      width: 100%;
    }
    .foreground {
      width: 100%;
    }
    .step {
      width: 100%;
      padding: 0;
      margin: 50vh auto;
    }
  }
  @media (max-width: 467px) {
    .step:nth-of-type(9) {
      display: none;
    }
  }
</style>
