<script>
  import * as d3 from 'd3';

  import Title from './Sections/Title.svelte'
  import Intro from './Sections/Intro.svelte'
  import Section1 from './Sections/Section1.svelte'
  import Section2 from './Sections/Section2.svelte'
  import Section3 from './Sections/Section3.svelte'
  import Section4 from './Sections/Section4.svelte'
  import Methodology from './Sections/Methodology.svelte'

  export let moonAccidentsD = [];
  export let mercuryDelaysD = [];
  export let mercuryRetrogradeD = [];
  export let mercuryAccidentsD = [];

  let w, h;

  
  let promise = getData();
  async function getData() {
    moonAccidentsD = await d3.csv("data/moon_accidents.csv")
    mercuryDelaysD = await d3.csv("data/mercury_delays.csv")
    mercuryRetrogradeD = await d3.csv("data/retrograde.csv")
    mercuryAccidentsD = await d3.csv("data/mercury_accidents.csv")
  }

</script>

<svelte:window bind:innerWidth="{w}" bind:innerHeight="{h}"/>

<main>
  {#if w !== undefined}
    <Title />
    <Intro />
    {#await promise then data} 
      <Section1 data={moonAccidentsD} width={w} height={h}/>
      <Section2 data={mercuryDelaysD} width={w} height={h}/>
      <Section3 data={mercuryRetrogradeD}/>
      <Section4 data={mercuryAccidentsD} width={w} height={h}/>
    {/await}
    <Methodology />
  {/if}
</main>

<style>
</style>