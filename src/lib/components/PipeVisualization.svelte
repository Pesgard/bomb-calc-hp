<script lang="ts">
	import { ArrowUp, ArrowDown } from "lucide-svelte";

	// Props for pipe visualization with default values to prevent null errors
	let { alturaInicial = $bindable(0), alturaFinal = $bindable(0), longitud = $bindable(0), diametro = $bindable(0), caudal = $bindable(0) } = $props();

	// Calculate pipe properties for visualization with null checks
	const pipeScale = $derived(Math.min((diametro || 0) / 10, 50));
	const heightDiff = $derived((alturaFinal || 0) - (alturaInicial || 0));
	const angle = $derived(Math.atan2(heightDiff, longitud || 0) * (180 / Math.PI));

	// Calculate flow indicators with null check
	const flowSpeed = $derived((caudal || 0) > 0 ? Math.min(Math.max((caudal || 0) * 1000, 0.5), 2) : 0);
</script>

<div class="relative w-full h-[400px] bg-gray-50 rounded-lg p-4">
	<!-- Pipe Visualization -->
	<div class="absolute inset-0 flex items-center justify-center">
		<div class="relative bg-blue-200 rounded-lg transform-origin-center transition-all duration-300" style="width: {Math.min((longitud || 0) * 2, 300)}px; height: {pipeScale}px; transform: rotate({angle}deg);">
			<!-- Flow Indicators -->
			{#if (caudal || 0) > 0}
				<div class="absolute inset-0 flex items-center justify-around overflow-hidden">
					{#each Array(3) as _, i}
						<ArrowUp class="text-blue-600 transform -rotate-90 transition-transform animate-pulse" style="animation-duration: {1 / flowSpeed}s; animation-delay: {i * 0.3}s" size={pipeScale * 0.8} />
					{/each}
				</div>
			{/if}
		</div>
	</div>

	<!-- Height Indicators -->
	<div class="absolute left-4 h-full flex flex-col justify-between">
		<div class="flex items-center gap-2">
			<ArrowUp class="text-gray-600" />
			<span class="text-sm">{(alturaFinal || 0).toFixed(1)}m</span>
		</div>
		<div class="flex items-center gap-2">
			<ArrowDown class="text-gray-600" />
			<span class="text-sm">{(alturaInicial || 0).toFixed(1)}m</span>
		</div>
	</div>

	<!-- Information Panel -->
	<div class="absolute bottom-4 right-4 bg-white p-3 rounded-lg shadow-sm">
		<p class="text-sm font-medium">Diámetro: {(diametro || 0).toFixed(2)}mm</p>
		<p class="text-sm font-medium">Longitud: {longitud || 0}m</p>
		<p class="text-sm font-medium">Caudal: {((caudal || 0) * 1000).toFixed(3)}L/s</p>
	</div>
</div>
