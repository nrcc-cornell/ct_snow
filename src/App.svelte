<script lang="ts">
  import * as L from 'leaflet';
  import Leaflet from './lib/map/Leaflet.svelte';
  import TileLayer from './lib/map/TileLayer.svelte';
  import GeoJson from './lib/map/GeoJson.svelte';
  import Tooltip from './lib/map/Tooltip.svelte';
  import Popup from './lib/map/Popup.svelte';
  import { onMount } from "svelte";
  
  import Chart from './lib/Chart.svelte'
  
  let snow_stns = []
  let selectedStns = []
  let map: Leaflet
  const initialBounds = L.latLngBounds([40.99194,-73.72618],[42.0494,-71.78796]);
  let loaded = false;

	function resizeMap() {
	  if(map) { map.invalidateSize(); }
  }
	
	function resetMapView() {
		map.setView(initialView, 5);
	}

  function stnClick(e: CustomEvent<any>,stn: any) {
    if (e.detail.originalEvent.shiftKey) {
      selectedStns.push(stn);
      selectedStns = selectedStns
    } else {
      selectedStns = [stn]
    }
  }


  onMount(async () => {
    snow_stns = await fetch("https://data.rcc-acis.org/MultiStnData",{
        method: 'POST',
        headers: {'Content-Type': 'application/json'},
        body: JSON.stringify({ state: 'CT', date: '2022-05', elems: [
          {name: 'snow', interval: [1,0], duration: 12, reduce: {reduce: 'sum', add: 'mcnt'}, add: ['mcnt'], maxmissing: 180}]})
        })
  .then((response) => response.json())
  .then((res) => res.data.sort((a: { data: number[][]; },b: { data: number[][]; }): boolean => a.data[0][1] > b.data[0][1]))
  .then((d) => {
  		// console.log(d)
      selectedStns.push(d[0])
      selectedStns.push(d[1])
      selectedStns = selectedStns
      return d
    })
  .catch(error => {
      console.log(error);
      return [];
    });
  });
  
  </script>

<style>
  #container {
    width: 100%;
    height: 600px;
    display: grid;
    grid-template:
      'map' auto;
  }
</style>

<svelte:window on:resize={resizeMap} on:load={() => (loaded = true)} />
<div style="width: 1000px; height: 10px;" />
<h1>Connecticut Snowfall</h1>

<div class="w-screen h-screen" id="container">
  <div style="grid-area:map;--dash-length:18">
    <!-- Show the map only once the window has loaded, so that Leaflet gets the sizing right. -->
    {#if loaded || document.readyState === 'complete'}
      <Leaflet bind:this={map} bounds={initialBounds}>
        <TileLayer
          url={"https://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}@2x.png"} 
          options = {{attribution: 'Data Source: <a href="https://www.rcc-acis.org/">ACIS</a> | &copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors'}}
        />
        {#each snow_stns as stn }
          {#if stn.meta.ll && stn.data[0][1] < 50 }
            <GeoJson
            geojson={{geometry: {type: "Point",coordinates: [stn.meta.ll[0],stn.meta.ll[1]]},type: "Feature"}}
            fillOpacity={0.8}
            fillColor="blue"
            color="blue"
            on:click={ (e)=> stnClick(e,stn) }
            >
            <Tooltip>{stn.meta.name}</Tooltip>
            </GeoJson>
          {/if}
        {/each}
      </Leaflet>
    {/if}
  </div>
</div>
<main>
  {#each selectedStns as station}
  <Chart station={station} />
  {/each}

</main>
