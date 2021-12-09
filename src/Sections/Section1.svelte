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

<Scroller top="{0}" bottom="{1}" threshold="{0.5}" bind:index bind:offset bind:progress>
  <div class="scroller background" slot="background">
    <svg {width} {height}>
      <text
        transform="translate({margin.l}, 50)"
        class="chart-title"
        fill="white">
        Moon phases and fatal accidents
      </text>
      <g class="legend" transform="translate({margin.l}, 100)" opacity="{index >= 2 ? 1 : 0}">
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
          Between November 1966 and 2021, there were 24,714 private and commercial flight crashes within the United States and its territories, according to an NTSB [aviation accident dataset/database/query]. 
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
          The most lethal crashes, four with more than 200 fatalities, occurred between the ‘70s and early 2000s. 
        </p>
        <p>
          Three of these four occurred between the new and earliest signs of waxing crescent, when the moon grows from 0% to 25% illumination.
        </p>
      </div>
    </section>
    <section data-section-id="4" class="step">
      <div class="step-text">
        <p>
          The other occurred in close proximity to the new moon, but during the final transition from waning crescent back to its origin.
        </p>
        <p>
          The night sky is darker during these time periods in comparison to the light provided by the full moon – but most crashes occurred during the daytime. 
        </p>
      </div>
    </section>
  </div>
</Scroller>

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
  <h2>
    A corporate pilot’s input
  </h2>
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
