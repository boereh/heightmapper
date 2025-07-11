<script lang="ts">
	import { map as createMap, Map, tileLayer } from 'leaflet';
	import { onMount } from 'svelte';
	import 'leaflet/dist/leaflet.css';
	import '@unocss/reset/tailwind.css';
	import 'virtual:uno.css';

	const TOPO_URL = 'https://api.opentopodata.org/v1/aster30m?locations=';
	const INCREMENTS = { latitude: 0.000269, longitude: 0.000326 };

	let center = $state({ latitude: 36.976524, longitude: -86.456017 });
	let blocks = $state({ horizontal: 0, vertical: 0 });
	let size = $state({ width: 512, height: 512 });
	let interpolation = $state(0);
	let updated = $state(Date.now());
	let map = $state<Map>();

	async function updation() {
		if (!map) return;

		center.latitude = map.getBounds().getCenter().lat;
		center.longitude = map.getBounds().getCenter().lng;
	}

	onMount(() => {
		if (typeof window === 'undefined') return;

		map = createMap('map', {
			center: [36.976524, -86.45601],
			zoom: 13,
			layers: [
				tileLayer(
					'https://server.arcgisonline.com/ArcGIS/rest/services/World_Topo_Map/MapServer/tile/{z}/{y}/{x}',
					{
						attribution: 'Tiles &copy; Esri'
					}
				)
			]
		});

		map.addEventListener('moveend', updation);
		map.addEventListener('resize', updation);
	});

	$effect(() => {
		if (!map) return;
		map.setView([center.latitude, center.longitude], map.getZoom());
	});
</script>

<div class="p-8">
	<div class="space-y-8 items-center max-w-7xl mx-auto">
		<div
			class="rounded-lg border border-neutral-200 select-none aspect-video w-full"
			id="map"
		></div>

		<div class="w-full grid grid-cols-4 gap-4">
			<div>
				<p class="mb-2 text-blue font-semibold">Latitude</p>
				<input
					bind:value={center.latitude}
					class="border rounded px-2 h-10 outline-none w-full"
					type="number"
					min="-90"
					max="90"
					step="0.000001"
				/>
			</div>

			<div>
				<p class="mb-2 text-blue font-semibold">Longitude</p>
				<input
					bind:value={center.longitude}
					class="border rounded px-2 h-10 outline-none w-full"
					type="number"
					min="0"
					step="0.000001"
				/>
			</div>

			<div>
				<p class="mb-2 text-blue font-semibold">Width (px)</p>
				<input
					bind:value={size.width}
					class="border rounded px-2 h-10 outline-none w-full"
					type="number"
					min="0"
					step="1"
				/>
			</div>

			<div>
				<p class="mb-2 text-blue font-semibold">Height (px)</p>
				<input
					bind:value={size.height}
					class="border rounded px-2 h-10 outline-none w-full"
					type="number"
					min="0"
					step="1"
				/>
			</div>

			<div>
				<p class="mb-2 text-blue font-semibold">Veritcal blocks</p>
				<input
					bind:value={blocks.vertical}
					class="border rounded px-2 h-10 outline-none w-full"
					type="number"
					min="1"
					step="1"
				/>
			</div>

			<div>
				<p class="mb-2 text-blue font-semibold">Horizontal blocks</p>
				<input
					bind:value={blocks.horizontal}
					class="border rounded px-2 h-10 outline-none w-full"
					type="number"
					min="1"
					step="1"
				/>
			</div>

			<div>
				<p class="mb-2 text-blue font-semibold">Interpolation</p>
				<input
					bind:value={interpolation}
					class="border rounded px-2 h-10 outline-none w-full"
					type="number"
					min="1"
					step="1"
				/>
			</div>
		</div>

		<div
			class="aspect-video h-full flex-grow rounded-lg bg-neutral-100 border border-neutral-200"
			id="map-output"
		>
			{JSON.stringify(blocks)}
		</div>
	</div>
</div>
