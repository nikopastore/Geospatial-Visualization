<!-- src/routes/+page.svelte -->

<script>
	// Import Mapbox GL JS
	import mapboxgl from 'mapbox-gl';
	import 'mapbox-gl/dist/mapbox-gl.css';
  
	import { onMount } from 'svelte';
  
	// Mapbox access token (replace 'YOUR_ACCESS_TOKEN' with your actual token)
	mapboxgl.accessToken = 'pk.eyJ1Ijoibmlrb3Bhc3RvcmUiLCJhIjoiY20zc2lpbzdoMGdqczJxcTB3czUzcTIxaiJ9.OxA2W627iC-9vWBa1_WHsg';
  
	let map;
  
	onMount(() => {
	  map = new mapboxgl.Map({
		container: 'map', // ID of the element to render the map in
		style: 'mapbox://styles/mapbox/streets-v12', // Map style
		center: [-71.0942, 42.3601], // [longitude, latitude] (MIT's location)
		zoom: 12, // Starting zoom level
	  });
	});
  </script>
  
  <style>
	@import url('$lib/global.css');
  
	/* Style for the map container */
	#map {
	  flex: 1;
	  height: 500px; /* Set a height for the map */
	}
  </style>
  
  <h1>Geospatial Visualization</h1>
  
  <p>
	Welcome to Geospatial Visualization, an immersive and interactive map visualization of bike traffic in the Boston area during different times of the day.
  </p>
  
  <div id="map"></div>
  