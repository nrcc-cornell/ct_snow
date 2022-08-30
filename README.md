# Connecticut Snowfall

Project is initialized via `npm create vite@latest ct_snow`

Select `svelte-ts`.

Finish setup.
```
cd ct_snow
npm i
```

## build, dev and run
```
npm run build
npm run preview
npm run dev
```

## Add functionality
* Replace template App.svelte with fetch code
* Replace the Counter component with Chart
    
  See `src/lib/Chart.svelte`
* Add dependencies

  `npm i @observablehq/plot d3-time`
* Add type definitions

  `npm i -D @types/d3-time`

## Add map based selector

Want to present the station location on a map and select stations.

  * tooltip to show station name on mouseover
  * only show stations that have a fair amount of data
  * Use the shift-key to show data for multiple stations 