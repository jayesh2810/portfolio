<script lang="ts">
	import { onMount } from 'svelte';

	let canvas: HTMLCanvasElement;
	let k = $state(3);
	let running = $state(false);
	let step = $state(0);

	const W = 600;
	const H = 400;
	const POINT_COUNT = 120;
	const COLORS = ['#C45D3E', '#4682B4', '#649A5A', '#C8AA50', '#8B5E8B'];

	interface Point { x: number; y: number; cluster: number; }
	interface Centroid { x: number; y: number; }

	let points: Point[] = $state([]);
	let centroids: Centroid[] = $state([]);

	function generateData() {
		step = 0;
		running = false;
		const numBlobs = 3 + Math.floor(Math.random() * 3);
		const blobCenters = Array.from({ length: numBlobs }, () => ({
			x: 60 + Math.random() * (W - 120),
			y: 60 + Math.random() * (H - 120)
		}));

		points = Array.from({ length: POINT_COUNT }, () => {
			const blob = blobCenters[Math.floor(Math.random() * numBlobs)];
			return {
				x: blob.x + (Math.random() - 0.5) * 140,
				y: blob.y + (Math.random() - 0.5) * 140,
				cluster: -1
			};
		});
		centroids = [];
		drawCanvas();
	}

	function initCentroids() {
		centroids = Array.from({ length: k }, () => ({
			x: 40 + Math.random() * (W - 80),
			y: 40 + Math.random() * (H - 80)
		}));
	}

	function assignClusters(): boolean {
		let changed = false;
		for (const p of points) {
			let minDist = Infinity;
			let best = 0;
			for (let c = 0; c < centroids.length; c++) {
				const dx = p.x - centroids[c].x;
				const dy = p.y - centroids[c].y;
				const d = dx * dx + dy * dy;
				if (d < minDist) { minDist = d; best = c; }
			}
			if (p.cluster !== best) { p.cluster = best; changed = true; }
		}
		return changed;
	}

	function updateCentroids() {
		for (let c = 0; c < centroids.length; c++) {
			const members = points.filter((p) => p.cluster === c);
			if (members.length === 0) continue;
			centroids[c] = {
				x: members.reduce((s, p) => s + p.x, 0) / members.length,
				y: members.reduce((s, p) => s + p.y, 0) / members.length
			};
		}
	}

	function drawCanvas() {
		if (!canvas) return;
		const ctx = canvas.getContext('2d')!;
		const dpr = typeof devicePixelRatio !== 'undefined' ? devicePixelRatio : 1;
		canvas.width = W * dpr;
		canvas.height = H * dpr;
		ctx.scale(dpr, dpr);
		ctx.clearRect(0, 0, W, H);

		for (const p of points) {
			ctx.beginPath();
			ctx.arc(p.x, p.y, 4, 0, Math.PI * 2);
			ctx.fillStyle = p.cluster >= 0 ? COLORS[p.cluster % COLORS.length] : '#B0A898';
			ctx.globalAlpha = 0.75;
			ctx.fill();
			ctx.globalAlpha = 1;
		}

		for (let c = 0; c < centroids.length; c++) {
			const ct = centroids[c];
			ctx.beginPath();
			ctx.arc(ct.x, ct.y, 8, 0, Math.PI * 2);
			ctx.fillStyle = COLORS[c % COLORS.length];
			ctx.fill();
			ctx.strokeStyle = '#fff';
			ctx.lineWidth = 2.5;
			ctx.stroke();

			ctx.beginPath();
			ctx.moveTo(ct.x - 5, ct.y);
			ctx.lineTo(ct.x + 5, ct.y);
			ctx.moveTo(ct.x, ct.y - 5);
			ctx.lineTo(ct.x, ct.y + 5);
			ctx.strokeStyle = '#fff';
			ctx.lineWidth = 2;
			ctx.stroke();
		}
	}

	async function runClustering() {
		if (running) return;
		running = true;
		step = 0;

		for (const p of points) p.cluster = -1;
		initCentroids();
		drawCanvas();
		await sleep(400);

		for (let iter = 0; iter < 20; iter++) {
			const changed = assignClusters();
			points = [...points];
			step = iter + 1;
			drawCanvas();
			await sleep(500);

			updateCentroids();
			centroids = [...centroids];
			drawCanvas();
			await sleep(400);

			if (!changed) break;
		}

		running = false;
	}

	function sleep(ms: number): Promise<void> {
		return new Promise((r) => setTimeout(r, ms));
	}

	onMount(() => {
		generateData();
	});
</script>

<div class="playground">
	<div class="playground__controls">
		<div class="playground__k-select">
			<span class="playground__label">K =</span>
			{#each [2, 3, 4, 5] as val}
				<button
					class="playground__k-btn"
					class:playground__k-btn--active={k === val}
					onclick={() => { k = val; }}
					disabled={running}
				>{val}</button>
			{/each}
		</div>
		<div class="playground__actions">
			<button class="playground__btn playground__btn--primary" onclick={runClustering} disabled={running}>
				{running ? `Step ${step}...` : 'Run Clustering'}
			</button>
			<button class="playground__btn" onclick={generateData} disabled={running}>
				Regenerate Data
			</button>
		</div>
	</div>

	<div class="playground__canvas-wrap">
		<canvas
			bind:this={canvas}
			class="playground__canvas"
			style="width: {W}px; height: {H}px;"
		></canvas>
	</div>

	<p class="playground__caption">
		K-means clustering on synthetic 2D data. One of the first algorithms I used for customer segmentation at C5i.
	</p>
</div>

<style>
	.playground {
		max-width: 640px;
		margin: 0 auto;
	}

	.playground__controls {
		display: flex;
		align-items: center;
		justify-content: space-between;
		flex-wrap: wrap;
		gap: 1rem;
		margin-bottom: 1.25rem;
	}

	.playground__k-select {
		display: flex;
		align-items: center;
		gap: 0.4rem;
	}

	.playground__label {
		font-family: var(--font-mono);
		font-size: 0.85rem;
		color: var(--color-muted);
		margin-right: 0.25rem;
	}

	.playground__k-btn {
		font-family: var(--font-mono);
		font-size: 0.8rem;
		width: 2rem;
		height: 2rem;
		border: 1px solid var(--color-border);
		border-radius: 6px;
		background: var(--color-card);
		color: var(--color-text);
		cursor: pointer;
		transition: all 0.2s var(--ease-smooth);
	}

	.playground__k-btn--active {
		background: var(--color-accent);
		color: white;
		border-color: var(--color-accent);
	}

	.playground__k-btn:disabled {
		opacity: 0.5;
		cursor: not-allowed;
	}

	.playground__actions {
		display: flex;
		gap: 0.5rem;
	}

	.playground__btn {
		font-family: var(--font-mono);
		font-size: 0.8rem;
		padding: 0.5rem 1rem;
		border: 1px solid var(--color-border);
		border-radius: 6px;
		background: var(--color-card);
		color: var(--color-text);
		cursor: pointer;
		transition: all 0.2s var(--ease-smooth);
	}

	.playground__btn:hover:not(:disabled) {
		border-color: var(--color-accent);
		color: var(--color-accent);
	}

	.playground__btn--primary {
		background: var(--color-accent);
		color: white;
		border-color: var(--color-accent);
	}

	.playground__btn--primary:hover:not(:disabled) {
		background: #b34e30;
		color: white;
	}

	.playground__btn:disabled {
		opacity: 0.5;
		cursor: not-allowed;
	}

	.playground__canvas-wrap {
		background: var(--color-card);
		border: 1px solid var(--color-border);
		border-radius: 8px;
		padding: 1rem;
		display: flex;
		justify-content: center;
		overflow: hidden;
	}

	.playground__canvas {
		display: block;
		max-width: 100%;
		height: auto;
	}

	.playground__caption {
		font-family: var(--font-mono);
		font-size: 0.75rem;
		color: var(--color-muted);
		text-align: center;
		margin-top: 1rem;
		line-height: 1.6;
	}
</style>
