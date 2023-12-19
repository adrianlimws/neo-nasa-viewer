<script>
	import '../app.css';
	import { T, Canvas } from '@threlte/core';
	import { Grid, OrbitControls, useTexture } from '@threlte/extras';
	import { onMount } from 'svelte';
	import { writable } from 'svelte/store';
	import { TextureLoader } from 'three';

	// Create a writable store for neo
	const neoStore = writable(1);
	// const earthTexture = useLoader(TextureLoader).load('/assets/8k_earth_daymap.jpg');

	export let neo;
	let neos = [];
	let currentNeoIndex = 0;

	async function fetchData() {
		try {
			// console.log('Fetching data for neo:', neo);
			const response = await fetch(
				`https://api.nasa.gov/neo/rest/v1/neo/browse?api_key=Y539x3gNFZbriWjlPKavmFxgZojJYtHxFZcoQ1Ku`
			);
			const responseData = await response.json();
			// Extract the neos array from the response
			neos = responseData.near_earth_objects || [];
			// Handle data as needed
			// console.log('Fetched data:', neos);
			return neos;
		} catch (error) {
			console.error('Error fetching data:', error);
		}
	}

	// Subscribe to the sol store and fetch data when it changes
	$: {
		neo = $neoStore;
	}
	let asteroid;
	// Fetch data when the component is mounted
	onMount(() => {
		console.log('Component mounted with neo:', neo);
		fetchData();
	});
	function changeNeoName(increment) {
		currentNeoIndex += increment;

		if (currentNeoIndex >= neos.length) {
			currentNeoIndex = 0;
		} else if (currentNeoIndex < 0) {
			currentNeoIndex = neos.length - 1;
		}
	}
</script>

<main class="flex items-stretch">
	<div class="basis-1/4 p-2 border-r-2 border-slate-800 text-center">
		{#if neos.length > 0}
			<div class="badge badge-md">
				ID: {neos[currentNeoIndex].id}
			</div>
			<div class="badge badge-success badge-md hover:bg-green-500">
				<a href={neos[currentNeoIndex].nasa_jpl_url}>NASA Small Lookup </a>
				<span class="inline-flex items-center ms-2 justify-center">
					<svg
						class="w-2.5 h-2.5 text-gray-800 dark:text-white"
						aria-hidden="true"
						xmlns="http://www.w3.org/2000/svg"
						fill="currentColor"
						viewBox="0 0 20 20"
					>
						<path
							d="m7.164 3.805-4.475.38L.327 6.546a1.114 1.114 0 0 0 .63 1.89l3.2.375 3.007-5.006ZM11.092 15.9l.472 3.14a1.114 1.114 0 0 0 1.89.63l2.36-2.362.38-4.475-5.102 3.067Zm8.617-14.283A1.613 1.613 0 0 0 18.383.291c-1.913-.33-5.811-.736-7.556 1.01-1.98 1.98-6.172 9.491-7.477 11.869a1.1 1.1 0 0 0 .193 1.316l.986.985.985.986a1.1 1.1 0 0 0 1.316.193c2.378-1.3 9.889-5.5 11.869-7.477 1.746-1.745 1.34-5.643 1.01-7.556Zm-3.873 6.268a2.63 2.63 0 1 1-3.72-3.72 2.63 2.63 0 0 1 3.72 3.72Z"
						/>
					</svg>
				</span>
			</div>
			{#if neos[currentNeoIndex].is_potentially_hazardous_asteroid}
				<div class="badge badge-warning badge-md">
					<span>Potentially Hazardous Asteroid</span>
				</div>
			{:else}
				<div class="badge badge-info badge-md">
					<span>Non Hazardous Asteroid</span>
				</div>
			{/if}

			<div class="">
				<p class="font-bold text-3xl text-white py-2">{neos[currentNeoIndex].name}</p>
			</div>

			<div class="stats stats-vertical lg:stats-horizontal">
				<div class="stat">
					<div class="stat-title">Min. Diameter</div>
					<div class="stat-value">
						{neos[currentNeoIndex].estimated_diameter.kilometers.estimated_diameter_min.toFixed(1)} km
					</div>
					<div class="stat-desc">
						{neos[currentNeoIndex].estimated_diameter.miles.estimated_diameter_min.toFixed(1)} miles
					</div>
					<div class="stat-desc">
						{neos[currentNeoIndex].estimated_diameter.meters.estimated_diameter_min.toFixed(1)} meters
					</div>
					<div class="stat-desc">
						{neos[currentNeoIndex].estimated_diameter.feet.estimated_diameter_min.toFixed(1)} ft
					</div>
				</div>

				<div class="stat">
					<div class="stat-title">Max. Diameter</div>
					<div class="stat-value">
						{neos[currentNeoIndex].estimated_diameter.kilometers.estimated_diameter_max.toFixed(1)} km
					</div>
					<div class="stat-desc">
						{neos[currentNeoIndex].estimated_diameter.miles.estimated_diameter_max.toFixed(1)} miles
					</div>
					<div class="stat-desc">
						{neos[currentNeoIndex].estimated_diameter.meters.estimated_diameter_max.toFixed(1)} meters
					</div>
					<div class="stat-desc">
						{neos[currentNeoIndex].estimated_diameter.feet.estimated_diameter_max.toFixed(1)} ft
					</div>
				</div>
			</div>

			<div class="overflow-x-auto text-white">
				<table class="table">
					<tbody>
						<tr>
							<th>First Observed:</th>
							<td>
								{new Date(
									neos[currentNeoIndex].orbital_data.first_observation_date
								).toLocaleDateString('en-US', {
									weekday: 'long', // 'long', 'short', or 'narrow'
									month: 'long', // 'long', 'short', or 'numeric'
									day: 'numeric',
									year: 'numeric'
								})}</td
							>
						</tr>
						<tr>
							<th>Last Observed:</th>
							<td
								>{new Date(
									neos[currentNeoIndex].orbital_data.last_observation_date
								).toLocaleDateString('en-US', {
									weekday: 'long', // 'long', 'short', or 'narrow'
									month: 'long', // 'long', 'short', or 'numeric'
									day: 'numeric',
									year: 'numeric'
								})}</td
							>
						</tr>
						<!-- row 3 -->
						<tr>
							<th>3</th>
							<td>Brice Swyre</td>
						</tr>
					</tbody>
				</table>
			</div>
		{:else}
			<p>Loading...</p>
		{/if}
	</div>

	<div class="canvas-container">
		<Canvas>
			<T.PerspectiveCamera makeDefault position={[0, 40, 40]} fov={35}>
				<OrbitControls
					autoRotate
					enableZoom={true}
					enableDamping
					autoRotateSpeed={0.1}
					target.y={1.5}
				/>
			</T.PerspectiveCamera>

			<T.DirectionalLight intensity={1} position.x={5} position.y={10} />
			<T.AmbientLight intensity={1} />

			<Grid
				position.y={0}
				cellColor="#fff"
				sectionColor="#fff"
				sectionThickness={0}
				fadeDistance={100}
				cellSize={2}
			/>
			<T.Mesh position={[0, 5, 0]}>
				<T.SphereGeometry
					args={[
						5,
						neos[currentNeoIndex].estimated_diameter.miles.estimated_diameter_min.toFixed(2),
						neos[currentNeoIndex].estimated_diameter.miles.estimated_diameter_max.toFixed(2)
					]}
				/>
				<T.MeshStandardMaterial color="red" roughness={8} metalness={0} />
			</T.Mesh>

			{#await useTexture('./lib/textures/earth.jpg') then earthTexture}
				<T.Mesh position={[-75, 5, 0]}>
					<T.SphereGeometry args={[62, 64, 64]} />
					<T.MeshStandardMaterial color="#0e2e53" map={earthTexture} />
				</T.Mesh>
			{/await}
		</Canvas>
	</div>

	<div class="btm-nav border-t-2 border-black">
		<button class="bg-black text-white hover:bg-slate-700" on:click={() => changeNeoName(-1)}>
			<svg
				class="w-6 h-6 text-white dark:text-white"
				aria-hidden="true"
				xmlns="http://www.w3.org/2000/svg"
				fill="none"
				viewBox="0 0 8 14"
			>
				<path
					stroke="currentColor"
					stroke-linecap="round"
					stroke-linejoin="round"
					stroke-width="2"
					d="M7 1 1.3 6.326a.91.91 0 0 0 0 1.348L7 13"
				/>
			</svg>
			<span class="btm-nav-label">Previous NEO</span>
		</button>

		<button class="bg-black text-white hover:bg-slate-700" on:click={() => changeNeoName(1)}>
			<svg
				class="w-6 h-6 text-white dark:text-white"
				aria-hidden="true"
				xmlns="http://www.w3.org/2000/svg"
				fill="none"
				viewBox="0 0 8 14"
			>
				<path
					stroke="currentColor"
					stroke-linecap="round"
					stroke-linejoin="round"
					stroke-width="2"
					d="m1 13 5.7-5.326a.909.909 0 0 0 0-1.348L1 1"
				/>
			</svg>
			<span class="btm-nav-label">Next NEO</span>
		</button>
	</div>
</main>

<style>
	:global(body) {
		margin: 0;
		background: rgb(13, 19, 32);
		background: linear-gradient(180deg, rgb(0, 0, 0) 0%, rgbrgb(22, 22, 22) 0%);
	}
	.canvas-container {
		width: 100%;
		height: 100vh;
		background: url('$lib/textures/8k_stars.jpg');
		background-repeat: no-repeat;
		background-size: 100% 100%;
	}
</style>
