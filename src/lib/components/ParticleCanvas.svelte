<script lang="ts">
	import { onMount } from 'svelte';

	let canvas: HTMLCanvasElement;
	let animationId: number;
	let isVisible = true;

	interface Particle {
		x: number;
		y: number;
		vx: number;
		vy: number;
		cluster: number;
		cx: number;
		cy: number;
		radius: number;
	}

	const clusterColors = [
		'rgba(196, 93, 62, 0.4)',   // terracotta
		'rgba(70, 130, 180, 0.4)',  // blue
		'rgba(100, 160, 90, 0.35)', // green
		'rgba(200, 170, 80, 0.4)'   // gold
	];

	const lineColors = [
		'rgba(196, 93, 62, ',
		'rgba(70, 130, 180, ',
		'rgba(100, 160, 90, ',
		'rgba(200, 170, 80, '
	];

	onMount(() => {
		const ctx = canvas.getContext('2d')!;
		let w: number, h: number;
		let particles: Particle[] = [];

		function resize() {
			w = canvas.clientWidth;
			h = canvas.clientHeight;
			canvas.width = w * devicePixelRatio;
			canvas.height = h * devicePixelRatio;
			ctx.scale(devicePixelRatio, devicePixelRatio);
		}

		function init() {
			resize();
			particles = [];
			const centers = [
				{ x: w * 0.25, y: h * 0.35 },
				{ x: w * 0.7, y: h * 0.3 },
				{ x: w * 0.4, y: h * 0.7 },
				{ x: w * 0.75, y: h * 0.65 }
			];

			for (let i = 0; i < 80; i++) {
				const cluster = Math.floor(Math.random() * 4);
				const c = centers[cluster];
				particles.push({
					x: c.x + (Math.random() - 0.5) * w * 0.25,
					y: c.y + (Math.random() - 0.5) * h * 0.25,
					vx: (Math.random() - 0.5) * 0.3,
					vy: (Math.random() - 0.5) * 0.3,
					cluster,
					cx: c.x,
					cy: c.y,
					radius: Math.random() * 2 + 1.5
				});
			}
		}

		function draw() {
			if (!isVisible) {
				animationId = requestAnimationFrame(draw);
				return;
			}

			ctx.clearRect(0, 0, w, h);

			for (let i = 0; i < particles.length; i++) {
				const p = particles[i];
				const dx = p.cx - p.x;
				const dy = p.cy - p.y;
				p.vx += dx * 0.0002;
				p.vy += dy * 0.0002;
				p.vx *= 0.998;
				p.vy *= 0.998;
				p.x += p.vx;
				p.y += p.vy;

				for (let j = i + 1; j < particles.length; j++) {
					const q = particles[j];
					if (p.cluster !== q.cluster) continue;
					const ddx = p.x - q.x;
					const ddy = p.y - q.y;
					const dist = Math.sqrt(ddx * ddx + ddy * ddy);
					if (dist < 120) {
						const opacity = (1 - dist / 120) * 0.15;
						ctx.beginPath();
						ctx.strokeStyle = lineColors[p.cluster] + opacity + ')';
						ctx.lineWidth = 0.8;
						ctx.moveTo(p.x, p.y);
						ctx.lineTo(q.x, q.y);
						ctx.stroke();
					}
				}
			}

			for (const p of particles) {
				ctx.beginPath();
				ctx.arc(p.x, p.y, p.radius, 0, Math.PI * 2);
				ctx.fillStyle = clusterColors[p.cluster];
				ctx.fill();
			}

			animationId = requestAnimationFrame(draw);
		}

		const observer = new IntersectionObserver(
			([entry]) => { isVisible = entry.isIntersecting; },
			{ threshold: 0.1 }
		);
		observer.observe(canvas);

		window.addEventListener('resize', resize);
		init();
		draw();

		return () => {
			cancelAnimationFrame(animationId);
			window.removeEventListener('resize', resize);
			observer.disconnect();
		};
	});
</script>

<canvas bind:this={canvas} class="particle-canvas" aria-hidden="true"></canvas>

<style>
	.particle-canvas {
		position: absolute;
		inset: 0;
		width: 100%;
		height: 100%;
		pointer-events: none;
	}
</style>
