<script>
	import mapboxgl from 'mapbox-gl';
	import 'mapbox-gl/dist/mapbox-gl.css';
	import * as d3 from 'd3';
	import { onMount } from 'svelte';
  
	mapboxgl.accessToken = 'pk.eyJ1Ijoibmlrb3Bhc3RvcmUiLCJhIjoiY20zc2lpbzdoMGdqczJxcTB3czUzcTIxaiJ9.OxA2W627iC-9vWBa1_WHsg';
  
	let map;
	let stations = [];
	let timeFilter = -1; // Default to no filter
	let radiusScale;
	const csvFile = '/bluebikes-traffic-2024-03.csv'; // Path to your CSV file
  
	async function fetchData() {
	  const trips = await d3.csv(csvFile);
  
	  // Preprocess station data (assuming Lat and Long columns in CSV)
	  stations = d3.rollups(
		trips,
		(v) => v.length,
		(d) => d.start_station_id
	  ).map(([id, totalTraffic]) => {
		const exampleTrip = trips.find((t) => t.start_station_id === id);
		return {
		  id,
		  name: exampleTrip.start_station_name,
		  lat: +exampleTrip.start_station_lat,
		  long: +exampleTrip.start_station_long,
		  totalTraffic
		};
	  });
  
	  calculateRadiusScale();
	}
  
	function calculateRadiusScale() {
	  const maxTraffic = d3.max(stations, (d) => d.totalTraffic);
	  radiusScale = d3.scaleSqrt().domain([0, maxTraffic]).range([0, 25]);
	}
  
	function updateMarkers() {
	  if (!map) return;
  
	  const svg = d3.select('#map').select('svg');
	  svg.selectAll('*').remove();
  
	  const projection = (lat, lng) => map.project([lng, lat]);
  
	  svg
		.selectAll('circle')
		.data(stations)
		.enter()
		.append('circle')
		.attr('cx', (d) => projection(d.lat, d.long).x)
		.attr('cy', (d) => projection(d.lat, d.long).y)
		.attr('r', (d) => radiusScale(d.totalTraffic))
		.attr('fill', 'blue')
		.attr('opacity', 0.5);
	}
  
	onMount(() => {
	  fetchData().then(() => {
		map = new mapboxgl.Map({
		  container: 'map',
		  style: 'mapbox://styles/mapbox/streets-v12',
		  center: [-71.0589, 42.3601], // Boston
		  zoom: 13
		});
  
		map.on('load', () => {
		  const svg = d3
			.select(map.getCanvasContainer())
			.append('svg')
			.attr('width', map.getContainer().clientWidth)
			.attr('height', map.getContainer().clientHeight);
  
		  map.on('move', updateMarkers);
		  updateMarkers();
		});
	  });
	});
  </script>
  
  <style>
	@import '$lib/global.css';
  </style>
  
  <h1>🚴 LaneWatchers - Boston Bike Traffic</h1>
  <div id="map"></div>
  <div class="slider">
	<label for="time">Filter by time: </label>
	<input type="range" id="time" min="-1" max="1440" bind:value={timeFilter} />
  </div>
  