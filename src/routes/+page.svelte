<script>
	import { onMount } from 'svelte';
	import mapboxgl from 'mapbox-gl';
	import * as d3 from 'd3';
	import 'mapbox-gl/dist/mapbox-gl.css';
  
	mapboxgl.accessToken = 'pk.eyJ1Ijoibmlrb3Bhc3RvcmUiLCJhIjoiY20zc2lpbzdoMGdqczJxcTB3czUzcTIxaiJ9.OxA2W627iC-9vWBa1_WHsg'; // Replace with your Mapbox token
  
	let map;
	let stations = [];
	let trips = [];
	let timeFilter = -1;
	let mapViewChanged = 0;
  
	const STATIONS_CSV = '/bluebikes-stations.csv';
	const TRIPS_CSV = '/bluebikes-traffic-2024-03.csv';
	const BIKE_LANES_GEOJSON = '/Existing_Bike_Network_2022.geojson';
  
	async function loadData() {
	  // Load station data
	  stations = await d3.csv(STATIONS_CSV, (d) => ({
		id: d.Number,
		name: d.NAME,
		lat: +d.Lat,
		long: +d.Long,
		totalDocks: +d['Total Docks'],
	  }));
  
	  console.log('Stations:', stations);
  
	  // Load trip data
	  trips = await d3.csv(TRIPS_CSV, (d) => ({
		startStationId: d.start_station_id,
		endStationId: d.end_station_id,
		startedAt: new Date(d.started_at),
		endedAt: new Date(d.ended_at),
	  }));
  
	  console.log('Trips:', trips);
	}
  
	function getCoords(station) {
	  if (!map) return { cx: 0, cy: 0 };
	  const point = map.project([station.long, station.lat]);
	  return { cx: point.x, cy: point.y };
	}
  
	$: filteredStations = stations.map((station) => {
	  const arrivals = trips.filter(
		(trip) =>
		  trip.endStationId === station.id &&
		  (timeFilter === -1 || Math.abs(minutesSinceMidnight(trip.startedAt) - timeFilter) <= 60)
	  ).length;
  
	  const departures = trips.filter(
		(trip) =>
		  trip.startStationId === station.id &&
		  (timeFilter === -1 || Math.abs(minutesSinceMidnight(trip.startedAt) - timeFilter) <= 60)
	  ).length;
  
	  return {
		...station,
		arrivals,
		departures,
		totalTraffic: arrivals + departures,
	  };
	});
  
	function minutesSinceMidnight(date) {
	  return date.getHours() * 60 + date.getMinutes();
	}
  
	$: radiusScale = d3
	  .scaleSqrt()
	  .domain([0, d3.max(filteredStations, (station) => station.totalTraffic)])
	  .range([0, 20]);
  
	onMount(async () => {
	  await loadData();
  
	  map = new mapboxgl.Map({
		container: 'map',
		style: 'mapbox://styles/mapbox/streets-v11',
		center: [-71.0589, 42.3601], // Boston coordinates
		zoom: 12,
	  });
  
	  map.on('load', () => {
		// Add bike lanes
		map.addSource('bikeLanes', {
		  type: 'geojson',
		  data: BIKE_LANES_GEOJSON,
		});
  
		map.addLayer({
		  id: 'bikeLanes',
		  type: 'line',
		  source: 'bikeLanes',
		  layout: {},
		  paint: {
			'line-color': '#28a745',
			'line-width': 2,
			'line-opacity': 0.6,
		  },
		});
  
		map.on('move', () => {
		  mapViewChanged++;
		});
	  });
	});
  </script>
  
  <style>
	body {
	  font-family: Arial, sans-serif;
	  margin: 0;
	  padding: 0;
	}
  
	h1 {
	  text-align: center;
	  font-size: 2rem;
	  margin: 1rem 0;
	}
  
	#map {
	  width: 100%;
	  height: 65vh;
	  position: relative;
	}
  
	.filter-bar {
	  display: flex;
	  justify-content: center;
	  margin-top: 1rem;
	}
  
	svg {
	  position: absolute;
	  top: 0;
	  left: 0;
	  pointer-events: none;
	}
  
	circle {
	  fill: steelblue;
	  stroke: white;
	  stroke-width: 1;
	  fill-opacity: 0.6;
	  pointer-events: auto;
	}
  </style>
  
  <h1>LaneWatchers</h1>
  
  <div id="map">
	<svg>
	  {#key mapViewChanged}
		{#each filteredStations as station}
		  <circle
			{...getCoords(station)}
			r={radiusScale(station.totalTraffic)}
		  >
			<title>
			  {station.name}: {station.totalTraffic} trips ({station.arrivals} arrivals, {station.departures} departures)
			</title>
		  </circle>
		{/each}
	  {/key}
	</svg>
  </div>
  
  <div class="filter-bar">
	<label>
	  Filter by time:
	  <input type="range" min="-1" max="1440" step="1" bind:value={timeFilter} />
	  <time>
		{timeFilter === -1
		  ? 'Any time'
		  : new Date(0, 0, 0, 0, timeFilter).toLocaleString('en', { timeStyle: 'short' })}
	  </time>
	</label>
  </div>
  