<script lang="ts">
import * as Plot from "@observablehq/plot";
import { utcDay } from "d3-time";


export let snowStns = [];
let station = undefined;
let snow = undefined;
let points = [];

function calcPoints(s) {
  let total = Number(0)
  let res = s.data.map(d => {
    if (d[1] == 'M' || d[1] == 'T') {
      return {date: new Date(d[0]), snow: 0., total: total}
    } else {
      total = total + Number(d[1])
      return {date: new Date(d[0]), snow: Number(d[1]), total: total}
    }
  })
	return res
}
	
const update = async (stn) => {
	if (!stn) { return }
// 	console.log(stn)
	snow = await fetch("https://data.rcc-acis.org/StnData",{
      method: 'POST',
      headers: {'Content-Type': 'application/json'},
      body: JSON.stringify({ uid: stn.meta.uid, 
														sdate: [2021,11,1], edate: [2022,4,1], 
														elems: ["snow"] })
      })
			.then((response) => response.json())
			.catch(error => {
					console.log(error);
					return undefined;
				});
	points = calcPoints(snow);
}

$: update(station)

function myplot(node: HTMLDivElement) {
		node.appendChild(
				Plot.plot({
					grid: false,
					x: { label: "Date" },
					y: { label: "Snowfall (in)" },
					marks: [
						Plot.rectY(points,{x: "date", interval: utcDay, y:"snow"}),
						Plot.line(points,{x: "date", y:"total"})
					]
				})
		)
}


</script>

<form>
	<label for="stns">Select Station</label><select id="stns" bind:value={station}>
	{#each snowStns.slice(0,20) as stn }
		<option value={stn}>{stn.meta.name}</option>
	{/each}
  </select>
</form>
<h1>{station?.meta.name}</h1>
{#key points}
<div use:myplot ></div>
{/key}