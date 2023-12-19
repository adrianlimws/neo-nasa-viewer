<script>
	import '../app.css';
	import { T, Canvas } from '@threlte/core';
	import { ContactShadows, Float, Grid, OrbitControls } from '@threlte/extras';

	import { onMount } from 'svelte';
	import { writable } from 'svelte/store';
	// import { TextureLoader } from 'three';
	// import earthTexturePath from '$lib/textures/8k_earth_daymap.jpg';
	// const earthTexture = new TextureLoader().load(earthTexturePath);
	// Create a writable store for neo
	const neoStore = writable(1);

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

<main class="grid grid-cols-3 gap-4">
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

		<div class="p-2">
			<p class="font-bold text-3xl">Codename: {neos[currentNeoIndex].name}</p>
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
	{/if}
	<div class="canvas-container basis-1/2">
		<Canvas>
			<T.PerspectiveCamera makeDefault position={[-15, 5, 10]} fov={100}>
				<OrbitControls
					autoRotate
					enableZoom={true}
					enableDamping
					autoRotateSpeed={0.01}
					target.y={1.5}
				/>
			</T.PerspectiveCamera>

			<T.DirectionalLight intensity={1} position.x={5} position.y={10} />
			<T.AmbientLight intensity={1} />

			<Grid
				position.y={-0.001}
				cellColor="#000"
				sectionColor="#000"
				sectionThickness={0}
				fadeDistance={25}
				cellSize={2}
			/>

			<T.Mesh position={[0, 0, 0]}>
				<T.BoxGeometry
					args={[
						10,
						neos[currentNeoIndex].estimated_diameter.miles.estimated_diameter_min,
						neos[currentNeoIndex].estimated_diameter.miles.estimated_diameter_min
					]}
				/>
				<T.MeshStandardMaterial color="red" />
			</T.Mesh>
		</Canvas>
	</div>
</main>

<style>
	:global(body) {
		margin: 0;
	}

	.canvas-container {
		width: 100vw;
		height: 100vh;
		background: rgb(13, 19, 32);
		background: linear-gradient(180deg, rgb(184, 184, 184) 0%, rgb(246, 246, 246) 100%);
	}
</style>
