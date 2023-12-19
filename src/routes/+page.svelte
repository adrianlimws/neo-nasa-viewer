<script>
	import '../app.css';
	import { T, Canvas } from '@threlte/core';
	import { Grid, OrbitControls, useTexture } from '@threlte/extras';
	import { onMount } from 'svelte';
	import { writable } from 'svelte/store';
	import { TextureLoader } from 'three';

	// Create a writable store for neo
	const neoStore = writable(1);

	export let neo;
	let neos = [];
	let currentNeoIndex = 0;
	let earthTexture;

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

	// Fetch data when the component is mounted
	onMount(() => {
		console.log('Component mounted with neo:', neo);
		fetchData();
		// const earthTexture = useLoader(TextureLoader).load('$lib/textures/8k_earth_daymap.jpg');
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
	<div class="basis-1/4 p-2 border-r-2 border-slate-800">
		{#if neos.length > 0}
			<div class="badge badge-lg">
				ID: {neos[currentNeoIndex].id}
			</div>
			{#if neos[currentNeoIndex].is_potentially_hazardous_asteroid}
				<div class="badge badge-error badge-lg">
					<span>Potentially Hazardous Asteroid</span>
				</div>
			{:else}
				<div class="badge badge-success badge-lg">
					<span>Not Potentially Hazardous Asteroid</span>
				</div>
			{/if}
			<div class="">
				<p class="font-bold text-3xl">{neos[currentNeoIndex].name}</p>
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

			<T.Mesh position={[-75, 5, 0]}>
				<T.SphereGeometry args={[62, 64, 64]} />
				<T.MeshStandardMaterial color="#0e2e53" />
			</T.Mesh>
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
