<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import { browser } from '$app/environment';
	import { readable } from 'svelte/store';
	import type { LatLngExpression } from 'leaflet';

	let mapContainer: HTMLDivElement;
	let map: L.Map | any;

	const shapesData = [
		{
			shape_id: '10B-R01_shp',
			shape_pt_lat: -6.231627,
			shape_pt_lon: 106.882884,
			shape_pt_sequence: 0
		},
		{
			shape_id: '10B-R01_shp',
			shape_pt_lat: -6.231435,
			shape_pt_lon: 106.882945,
			shape_pt_sequence: 1
		},
		{
			shape_id: '10B-R01_shp',
			shape_pt_lat: -6.231143,
			shape_pt_lon: 106.882212,
			shape_pt_sequence: 2
		},
		{
			shape_id: '10B-R01_shp',
			shape_pt_lat: -6.231118,
			shape_pt_lon: 106.882226,
			shape_pt_sequence: 3
		},
		{
			shape_id: '10B-R01_shp',
			shape_pt_lat: -6.231422,
			shape_pt_lon: 106.882974,
			shape_pt_sequence: 4
		},
		{
			shape_id: '10B-R01_shp',
			shape_pt_lat: -6.231746,
			shape_pt_lon: 106.882872,
			shape_pt_sequence: 5
		},
		{
			shape_id: '10B-R01_shp',
			shape_pt_lat: -6.233022,
			shape_pt_lon: 106.882377,
			shape_pt_sequence: 6
		},
		{
			shape_id: '10B-R01_shp',
			shape_pt_lat: -6.233048,
			shape_pt_lon: 106.882366,
			shape_pt_sequence: 7
		},
		{
			shape_id: '10B-R01_shp',
			shape_pt_lat: -6.233062,
			shape_pt_lon: 106.882334,
			shape_pt_sequence: 8
		},
		{
			shape_id: '10B-R01_shp',
			shape_pt_lat: -6.232854,
			shape_pt_lon: 106.88187,
			shape_pt_sequence: 9
		},
		{
			shape_id: '10B-R01_shp',
			shape_pt_lat: -6.232687,
			shape_pt_lon: 106.881604,
			shape_pt_sequence: 10
		},
		{
			shape_id: '10B-R01_shp',
			shape_pt_lat: -6.232615,
			shape_pt_lon: 106.88137,
			shape_pt_sequence: 11
		}
		// Add all remaining shape points from shapes.txt here...
	];

	// GTFS trips.txt data (example)
	const tripsData = [
		{ route_id: '10B', trip_id: '10B-R01', shape_id: '10B-R01_shp' },
		{ route_id: '10B', trip_id: '10B-R02', shape_id: '10B-R02_shp' }
		// ... (add the rest of tripsData)
	];

	// Group shapes by shape_id
	const shapesMap: Record<string, LatLngExpression[]> = {};

	shapesData.forEach((shape) => {
		if (!shapesMap[shape.shape_id]) {
			shapesMap[shape.shape_id] = [];
		}
		shapesMap[shape.shape_id].push([shape.shape_pt_lat, shape.shape_pt_lon]);
	});

	// Filter trips to get the shape_id for each trip
	const tripShapes: LatLngExpression[][] = tripsData.map((trip) => {
		const shapeId = trip.shape_id;
		return shapesMap[shapeId] || [];
	});

	onMount(async () => {
		if (browser) {
			// Dynamically import Leaflet
			const L = (await import('leaflet')) as typeof import('leaflet');

			// Initialize the map
			map = L.map(mapContainer);

			// Add the tile layer with a light grey style
			L.tileLayer('https://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}.png', {
				attribution: '&copy; OpenStreetMap contributors &copy; CartoDB'
			}).addTo(map);

			const allCoordinates: LatLngExpression[] = [];

			// Add the shapes as polylines and collect all coordinates for bounds
			tripShapes.forEach((route) => {
				if (route.length > 0) {
					L.polyline(route, { color: 'green' }).addTo(map);
					allCoordinates.push(...route);
				}
			});

			// Fit the map to the bounds of all routes
			if (allCoordinates.length > 0) {
				const bounds = L.latLngBounds(allCoordinates);
				map.fitBounds(bounds);
			}
		}
	});

	onDestroy(() => {
		if (map) {
			map.remove();
		}
	});
</script>

<main>
	<div bind:this={mapContainer} style="height: 500px;"></div>
</main>

<style>
	@import 'leaflet/dist/leaflet.css';
</style>
