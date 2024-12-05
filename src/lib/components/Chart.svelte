<script lang="ts">
	// Update props to use svelte 5 $props rune
	let { data }: { data: { caudales: number[]; alturas: number[] } } = $props();

	let canvas: HTMLCanvasElement = $state();

	$effect(() => {
		if (!canvas || !data.caudales.length) return;

		const ctx = canvas.getContext('2d');
		if (!ctx) return;

		// Clear previous drawing
		ctx.clearRect(0, 0, canvas.width, canvas.height);

		// Set dimensions
		const padding = 70;
		const width = canvas.width - padding * 2;
		const height = canvas.height - padding * 2;

		// Find ranges
		const maxX = Math.max(...data.caudales);
		const maxY = Math.max(...data.alturas);

		// Draw axes
		ctx.beginPath();
		ctx.moveTo(padding, padding);
		ctx.lineTo(padding, canvas.height - padding);
		ctx.lineTo(canvas.width - padding, canvas.height - padding);
		ctx.stroke();

		// Draw curve
		ctx.beginPath();
		ctx.strokeStyle = 'blue';
		data.caudales.forEach((x, i) => {
			const xPos = padding + (x / maxX) * width;
			const yPos = canvas.height - padding - (data.alturas[i] / maxY) * height;

			if (i === 0) {
				ctx.moveTo(xPos, yPos);
			} else {
				ctx.lineTo(xPos, yPos);
			}
		});
		ctx.stroke();

		// Add labels
		ctx.fillStyle = 'black';
		ctx.font = '12px Arial';
		ctx.textAlign = 'right';
		ctx.fillText('Altura (m)', padding - 5, padding - 5);
		ctx.textAlign = 'center';
		ctx.fillText('Caudal (m³/s)', canvas.width / 2, canvas.height - 5);
	});
</script>

<div class="aspect-[16/9] w-full">
	<canvas bind:this={canvas} width="800" height="450" class="h-full w-full rounded border bg-white"
	></canvas>
</div>
