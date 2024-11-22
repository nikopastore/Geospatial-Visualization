<script>
  import mapboxgl from 'mapbox-gl';
  import 'mapbox-gl/dist/mapbox-gl.css';
  import { onMount } from 'svelte';

  mapboxgl.accessToken = 'pk.eyJ1Ijoibmlrb3Bhc3RvcmUiLCJhIjoiY20zc2lpbzdoMGdqczJxcTB3czUzcTIxaiJ9.OxA2W627iC-9vWBa1_WHsg';

  let map;

  onMount(() => {
    // Initialize the Mapbox map
    map = new mapboxgl.Map({
      container: 'map',
      style: 'mapbox://styles/mapbox/streets-v12',
      center: [-71.0942, 42.3601],
      zoom: 12,
    });

    // Add Boston bike lanes data source
    map.on('load', () => {
      map.addSource('boston_bike_lanes', {
        type: 'geojson',
        data: 'https://bostonopendata-boston.opendata.arcgis.com/datasets/boston::existing-bike-network-2022.geojson',
      });

      // Add a layer to visualize the bike lanes
      map.addLayer({
        id: 'boston_bike_lanes',
        type: 'line',
        source: 'boston_bike_lanes',
        paint: {
          'line-color': '#2ecc71', // Green color
          'line-width': 3,
          'line-opacity': 0.8,
        },
      });
    });
  });
</script>

<style>
  @import url('$lib/global.css');

  #map {
    width: 100%;
    height: 500px;
    border: 1px solid #ccc;
  }
</style>

<h1>Geospatial Visualization</h1>

<p>
  Welcome to Geospatial Visualization, an immersive and interactive map visualization of bike traffic in the Boston area during different times of the day.
</p>

<div id="map"></div>
