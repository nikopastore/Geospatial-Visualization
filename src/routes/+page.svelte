<script>
	import { onMount } from 'svelte';
	import mapboxgl from 'mapbox-gl';
	import * as d3 from 'd3';
	import 'mapbox-gl/dist/mapbox-gl.css';

	mapboxgl.accessToken = 'pk.eyJ1Ijoibmlrb3Bhc3RvcmUiLCJhIjoiY20zc2lpbzdoMGdqczJxcTB3czUzcTIxaiJ9.OxA2W627iC-9vWBa1_WHsg';

	let map;
	let stations = [];
	let trips = [];
	let filteredTrips = [];
	let timeFilter = -1; // Default filter: show all times

	const STATIONS_CSV = '/bluebikes-stations.csv';
	const TRIPS_CSV = '/bluebikes-traffic-2024-03.csv';
	const BIKE_LANES_GEOJSON = '/Existing_Bike_Network_2022.geojson';

	// Load data
	async function loadData() {
		stations = await d3.csv(STATIONS_CSV, (d) => ({
			id: d.Number,
			name: d.NAME,
			lat: +d.Lat,
			long: +d.Long,
			totalDocks: +d['Total Docks'],
		}));

		trips = await d3.csv(TRIPS_CSV, (d) => ({
			startStationId: d.start_station_id,
			endStationId: d.end_station_id,
			startedAt: new Date(d.started_at),
			endedAt: new Date(d.ended_at),
		}));

		console.log('Data loaded:', { stations, trips });
		filterTripsByTime();
	}

	// Utility function for minutes since midnight
	function minutesSinceMidnight(date) {
		return date.getHours() * 60 + date.getMinutes();
	}

	// Filter trips by time range
	function filterTripsByTime() {
    if (timeFilter === -1) {
        filteredTrips = trips;
    } else {
        const start = timeFilter - 60;
        const end = timeFilter + 60;

        filteredTrips = trips.filter((trip) => {
            const startedMinutes = minutesSinceMidnight(trip.startedAt);
            const endedMinutes = minutesSinceMidnight(trip.endedAt);
            return (
                (startedMinutes >= start && startedMinutes <= end) ||
                (endedMinutes >= start && endedMinutes <= end)
            );
        });
    }

	console.log('Filtered Trips:', filteredTrips);
    updateStationsOnMap(); // Update markers based on filtered data
}


	// Update station markers dynamically
	function updateStationsOnMap() {
    if (!map || !stations.length) return;

    const trafficMap = new Map();
    filteredTrips.forEach((trip) => {
        if (!trafficMap.has(trip.startStationId)) {
            trafficMap.set(trip.startStationId, { arrivals: 0, departures: 0 });
        }
        if (!trafficMap.has(trip.endStationId)) {
            trafficMap.set(trip.endStationId, { arrivals: 0, departures: 0 });
        }

        trafficMap.get(trip.startStationId).departures++;
        trafficMap.get(trip.endStationId).arrivals++;
    });

    // Clear existing markers
    document.querySelectorAll('.station-marker').forEach((el) => el.remove());

    // Add station markers dynamically
    stations.forEach((station) => {
        const traffic = trafficMap.get(station.id) || { arrivals: 0, departures: 0 };
        const totalTraffic = traffic.arrivals + traffic.departures;

        if (totalTraffic === 0) return; // Skip stations with no traffic

        const markerSize = Math.max(2, Math.sqrt(totalTraffic) * 0.5); // Adjust size dynamically

        const markerElement = document.createElement('div');
        markerElement.className = 'station-marker';
        markerElement.style.width = `${markerSize}px`;
        markerElement.style.height = `${markerSize}px`;
        markerElement.style.backgroundColor = 'steelblue';
        markerElement.style.border = '1px solid white';
        markerElement.style.borderRadius = '50%';
        markerElement.style.opacity = '0.6';
        markerElement.title = `${station.name}\nArrivals: ${traffic.arrivals}\nDepartures: ${traffic.departures}`;

        new mapboxgl.Marker(markerElement)
            .setLngLat([station.long, station.lat])
            .addTo(map);
    });
}


	// Initialize map and data
	onMount(async () => {
		await loadData();

		map = new mapboxgl.Map({
			container: 'map',
			style: 'mapbox://styles/mapbox/streets-v11',
			center: [-71.0589, 42.3601], // Boston
			zoom: 12,
		});

		map.on('load', () => {
			// Add bike lanes layer
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

			updateStationsOnMap();
		});

		$: filterTripsByTime(); // Update on filter change
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

	.station-marker {
		background-color: steelblue;
		border: 1px solid white;
		border-radius: 50%;
		pointer-events: none;
		opacity: 0.6;
	}

	.filter-bar {
		display: flex;
		justify-content: center;
		align-items: center;
		margin: 1rem;
	}

	.filter-bar time {
		display: inline-block;
		width: 100px;
		text-align: center;
	}
</style>

<h1>LaneWatchers</h1>

<div id="map"></div>

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
