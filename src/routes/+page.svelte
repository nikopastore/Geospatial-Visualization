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
		center: [-71.0942, 42.3601], // Boston area
		zoom: 12
	  });
  
	  // Add Boston bike lanes
	  map.on('load', () => {
		map.addSource('boston_bike_lanes', {
		  type: 'geojson',
		  data: 'https://bostonopendata-boston.opendata.arcgis.com/datasets/boston::existing-bike-network-2022.geojson'
		});
  
		map.addLayer({
		  id: 'boston_bike_lanes',
		  type: 'line',
		  source: 'boston_bike_lanes',
		  paint: {
			'line-color': '#2ecc71',
			'line-width': 3,
			'line-opacity': 0.8
		  }
		});
	  });
	});
  </script>
  
  <style>
	@import '$lib/global.css';
  
	h1 {
	  font-size: 2rem;
	  margin: 0.5rem 0;
	}
  
	p {
	  margin-bottom: 1rem;
	}
  </style>
  
  <h1>Geospatial Visualization</h1>
  <p>Explore bike lane usage in the Boston area with an interactive map.</p>
  
  <div id="map"></div>
  