<script>
  import * as d3 from 'd3';

  import Title from './Sections/Title.svelte'
  import Intro from './Sections/Intro.svelte'
  import Section1 from './Sections/Section1.svelte'
  import Section2 from './Sections/Section2.svelte'
  import Section3 from './Sections/Section3.svelte'
  import Section4 from './Sections/Section4.svelte'
  import Conclusion from './Sections/Conclusion.svelte'
  import Methodology from './Sections/Methodology.svelte'

  export let moonAccidentsD = [];
  export let mercuryDelaysD = [];
  export let mercuryRetrogradeD = [];
  // export let mercuryAccidentsD = [];

  
  let promise = getData();
  async function getData() {
    moonAccidentsD = await d3.csv("data/moon_accidents.csv")
    mercuryDelaysD = await d3.csv("data/mercury_delays.csv")
    mercuryRetrogradeD = await d3.csv("data/retrograde.csv")
  }

</script>

<main>
  <Title />
  <Intro />
  {#await promise then data} 
    <Section1 data={moonAccidentsD} />
    <Section2 data={mercuryDelaysD}/>
    <Section3 data={mercuryRetrogradeD}/>
    <Section4 />
  {/await}
  <Conclusion />
  <Methodology />
</main>

<style>
</style>