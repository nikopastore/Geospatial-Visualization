<script>
	import mapboxgl from 'mapbox-gl';
	import 'mapbox-gl/dist/mapbox-gl.css';
	import { onMount } from 'svelte';
  
	mapboxgl.accessToken = 'pk.eyJ1Ijoibmlrb3Bhc3RvcmUiLCJhIjoiY20zc2lpbzdoMGdqczJxcTB3czUzcTIxaiJ9.OxA2W627iC-9vWBa1_WHsg';
  
	let map;
  
	onMount(() => {
	  map = new mapboxgl.Map({
		container: 'map',
		style: 'mapbox://styles/mapbox/streets-v12',
		center: [-117.1625, 32.715], // Centered on downtown San Diego
		zoom: 12 // Zoom level to show downtown coastline through the 805
	  });
  
	  // Example: Adding a data source for lanes (update to your real data if available)
	  map.on('load', () => {
		map.addSource('bike_lanes', {
		  type: 'geojson',
		  data: 'https://example.com/san-diego-bike-lanes.geojson' // Replace with your dataset
		});
  
		map.addLayer({
		  id: 'bike_lanes',
		  type: 'line',
		  source: 'bike_lanes',
		  paint: {
			'line-color': '#ff5722', // Orange color for lanes
			'line-width': 3,
			'line-opacity': 0.8
		  }
		});
	  });
	});
  </script>
  
  <style>
	@import '$lib/global.css';
  </style>
  
  <h1>🚴 LaneWatchers</h1>
  <div id="map"></div>
  