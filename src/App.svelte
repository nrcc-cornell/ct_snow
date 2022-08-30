<script lang="ts">
  import { onMount } from "svelte";
  import Chart from './lib/Chart.svelte'
  
  let snow_stns = []
  onMount(async () => {
    snow_stns = await fetch("https://data.rcc-acis.org/MultiStnData",{
        method: 'POST',
        headers: {'Content-Type': 'application/json'},
        body: JSON.stringify({ state: 'CT', date: '2022-05', elems: [
          {name: 'snow', interval: [1,0], duration: 12, reduce: {reduce: 'sum', add: 'mcnt'}, add: ['mcnt'], maxmissing: 180}]})
        })
  .then((response) => response.json())
  .then((res) => res.data.sort((a: { data: number[][]; },b: { data: number[][]; }): boolean => a.data[0][1] > b.data[0][1]))
  .catch(error => {
      console.log(error);
      return [];
    });
  });
  
  </script>
  
  <main>
    <h1>Connecticut Snowfall</h1>

     {#if snow_stns.length > 0 }
        <Chart snowStns={snow_stns} />
    {/if}
  </main>
