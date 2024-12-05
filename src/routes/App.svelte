<script lang="ts">
	import Chart from '$lib/components/Chart.svelte';
	import PipeVisualization from '$lib/components/PipeVisualization.svelte';
	import { TUBERIAS } from '$lib/data.js';

	// Types for calculations
	type TuberiaData = {
		diametro: number;
		rugosidad: number;
	};

	// Form input states
	let longitud: number = $state(0);
	let alturaInicial: number = $state(0);
	let alturaFinal: number = $state(0);
	let presionInicial: number = $state(0);
	let densidad: number = $state(0);
	let caudal: number = $state(0);
	let tipoTuberia: string = $state('cedula_40');
	let diametroNominal: string = $state('1.0');

	// Results states
	let results: string = $state('');
	let chartData: { caudales: number[]; alturas: number[] } = $state({ caudales: [], alturas: [] });
	let showResults: boolean = $state(false);

	// Get current pipe diameter for visualization
	const currentDiameter = $derived(TUBERIAS[tipoTuberia][diametroNominal].diametro * 1000);

	/** Calculate pump specifications */
	const calcularBomba = () => {
		try {
			const datosTubo = TUBERIAS[tipoTuberia][diametroNominal];
			const D = datosTubo.diametro;
			const rugosidad = datosTubo.rugosidad;

			// Calculate area and velocity
			const A = (Math.PI * D ** 2) / 4;
			const V = caudal / A;

			// Reynolds number
			const viscosidad = 0.001; // Pa·s
			const Re = (densidad * V * D) / viscosidad;

			// Friction factor using Colebrook-White
			let f = 0.02;
			for (let i = 0; i < 100; i++) {
				const f_new = f - colebrookWhite(f, rugosidad, D, Re) / (-2 / f ** 1.5);
				if (Math.abs(f_new - f) < 1e-6) {
					f = f_new;
					break;
				}
				f = f_new;
			}

			// Calculate head loss and total dynamic head
			const g = 9.81;
			const hf = (f * (longitud / D) * V ** 2) / (2 * g);
			const HDT = alturaFinal - alturaInicial + hf;

			// Calculate powers
			const Ph = densidad * g * caudal * HDT;
			const eficiencia = 0.7;
			const BHP = Ph / eficiencia;
			const potenciaBombaKw = Math.ceil(BHP / 1000);
			const costoOperacion = potenciaBombaKw * 0.1 * 24;

			// Generate chart data
			const caudales = Array.from({ length: 10 }, (_, i) => caudal * (0.8 + 0.02 * i));
			const alturas = Array.from({ length: 10 }, (_, i) => HDT - 0.1 * i * HDT);

			chartData = { caudales, alturas };

			results =
				`Altura dinámica total (HDT): ${HDT.toFixed(2)} m\n` +
				`Potencia hidráulica: ${(Ph / 1000).toFixed(2)} kW\n` +
				`Potencia de freno (BHP): ${(BHP / 1000).toFixed(2)} kW\n` +
				`Potencia recomendada de la bomba: ${potenciaBombaKw} kW\n` +
				`Costo diario estimado de operación: $${costoOperacion.toFixed(2)} USD`;

			showResults = true;
		} catch (error) {
			results = `Error inesperado: ${error}`;
			showResults = false;
		}
	};

	/** Validate all inputs are positive numbers */
	const isValidInputs = () => {
		return [longitud, alturaInicial, alturaFinal, presionInicial, densidad, caudal].every(
			(value) => value > 0
		);
	};

	/** Colebrook-White equation */
	const colebrookWhite = (f: number, rugosidad: number, D: number, Re: number) => {
		return -2 * Math.log10(rugosidad / (3.7 * D) + 2.51 / (Re * Math.sqrt(f))) - 1 / Math.sqrt(f);
	};

	/** Handle form submission */
	const handleSubmit = (e: SubmitEvent) => {
		e.preventDefault();
		if (!isValidInputs()) {
			results = 'Error: Todas las entradas deben ser números positivos';
			showResults = true;
			return;
		}
		calcularBomba();
	};
</script>

<main class="min-h-screen bg-gray-100 px-4 py-8">
	<div class="mx-auto max-w-6xl space-y-6">
		<div class="grid grid-cols-1 gap-6 lg:grid-cols-3">
			<!-- Form Section -->
			<div class="rounded-lg bg-white p-6 shadow-md lg:col-span-2">
				<h1 class="mb-6 text-center text-2xl font-bold">Calculadora de Bomba BHP</h1>

				<form onsubmit={handleSubmit} class="grid grid-cols-1 gap-4 md:grid-cols-2">
					<div class="space-y-2">
						<label class="block text-sm font-medium"
							>Longitud (m)
							<input
								type="number"
								bind:value={longitud}
								class="w-full rounded border p-2"
								min="0"
								step="0.1"
							/>
						</label>
					</div>

					<div class="space-y-2">
						<label class="block text-sm font-medium"
							>Altura Inicial (m)
							<input
								type="number"
								bind:value={alturaInicial}
								class="w-full rounded border p-2"
								min="0"
								step="0.1"
							/>
						</label>
					</div>

					<div class="space-y-2">
						<label class="block text-sm font-medium"
							>Altura Final (m)
							<input
								type="number"
								bind:value={alturaFinal}
								class="w-full rounded border p-2"
								min="0"
								step="0.1"
							/>
						</label>
					</div>

					<div class="space-y-2">
						<label class="block text-sm font-medium"
							>Presión Inicial (Pa)
							<input
								type="number"
								bind:value={presionInicial}
								class="w-full rounded border p-2"
								min="0"
								step="100"
							/>
						</label>
					</div>

					<div class="space-y-2">
						<label class="block text-sm font-medium"
							>Densidad (kg/m³)
							<input
								type="number"
								bind:value={densidad}
								class="w-full rounded border p-2"
								min="0"
								step="0.1"
							/>
						</label>
					</div>

					<div class="space-y-2">
						<label class="block text-sm font-medium"
							>Caudal (m³/s)
							<input
								type="number"
								bind:value={caudal}
								class="w-full rounded border p-2"
								min="0"
								step="0.001"
							/>
						</label>
					</div>

					<div class="space-y-2">
						<label class="block text-sm font-medium"
							>Tipo de Tubería
							<select bind:value={tipoTuberia} class="w-full rounded border p-2">
								<option value="cedula_40">Cédula 40</option>
								<option value="cedula_80">Cédula 80</option>
							</select>
						</label>
					</div>

					<div class="space-y-2">
						<label class="block text-sm font-medium"
							>Diámetro Nominal
							<select bind:value={diametroNominal} class="w-full rounded border p-2">
								<option value="0.5">½"</option>
								<option value="0.75">¾"</option>
								<option value="1.0">1"</option>
								<option value="1.25">1¼"</option>
								<option value="1.5">1½"</option>
								<option value="2.0">2"</option>
							</select>
						</label>
					</div>

					<div class="md:col-span-2">
						<button
							type="submit"
							class="w-full rounded bg-blue-600 py-2 text-white transition hover:bg-blue-700"
							>Calcular</button
						>
					</div>
				</form>
			</div>

			<!-- Pipe Visualization Section -->
			<div class="rounded-lg bg-white p-6 shadow-md">
				<h2 class="mb-4 text-xl font-semibold">Visualización de Tubería</h2>
				<PipeVisualization
					{alturaInicial}
					{alturaFinal}
					{longitud}
					diametro={currentDiameter}
					{caudal}
				/>
			</div>
		</div>

		{#if showResults}
			<div class="space-y-4 rounded-lg bg-white p-6 shadow-md">
				<h2 class="text-xl font-semibold">Resultados</h2>
				<div class="whitespace-pre-line rounded bg-gray-50 p-4">
					{results}
				</div>
				<Chart data={chartData} />
			</div>
		{/if}
	</div>
</main>
