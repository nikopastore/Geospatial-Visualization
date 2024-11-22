<script>
  import mapboxgl from 'mapbox-gl';
  import 'mapbox-gl/dist/mapbox-gl.css';
  import { onMount } from 'svelte';

  // Your Mapbox access token (replace with your actual token)
  mapboxgl.accessToken = 'pk.eyJ1Ijoibmlrb3Bhc3RvcmUiLCJhIjoiY20zc2lpbzdoMGdqczJxcTB3czUzcTIxaiJ9.OxA2W627iC-9vWBa1_WHsg';

  let map;

  onMount(() => {
    map = new mapboxgl.Map({
      container: 'map', // ID of the div where the map will be rendered
      style: 'mapbox://styles/mapbox/streets-v12', // Default Mapbox style
      center: [-71.0942, 42.3601], // Longitude and Latitude (MIT's location)
      zoom: 12, // Initial zoom level
    });
  });
</script>

<style>
  @import url('$lib/global.css');

  #map {
    width: 100%; /* Full width of the parent container */
    height: 500px; /* Height of the map */
    border: 1px solid #ccc; /* Optional: Add a border for clarity */
  }
</style>

<h1>Geospatial Visualization</h1>

<p>
  Welcome to Geospatial Visualization, an immersive and interactive map visualization of bike traffic in the Boston area during different times of the day.
</p>

<div id="map"></div>
