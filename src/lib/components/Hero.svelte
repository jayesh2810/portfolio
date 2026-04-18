<script lang="ts">
	import { onMount } from 'svelte';
	import { base } from '$app/paths';
	import ParticleCanvas from './ParticleCanvas.svelte';

	let loaded = $state(false);

	onMount(() => {
		requestAnimationFrame(() => { loaded = true; });
	});
</script>

<section class="hero" id="home">
	<ParticleCanvas />

	<div class="hero__content" class:hero__content--visible={loaded}>
		<span class="hero__label">{'// data scientist & ml engineer'}</span>
		<h1 class="hero__name">Jayesh<span class="hero__dot">.</span></h1>
		<p class="hero__subtitle">
			I build AI systems that connect business problems to working solutions.<br />
			Currently exploring agentic architectures and LLM pipelines.
		</p>
		<div class="hero__links">
			<a href="{base}/Jayesh%20Bhadane%20-%20Data%20Scientist%20Resume.pdf" target="_blank" rel="noopener" class="hero__link">resume</a>
			<span class="hero__sep">·</span>
			<a href="https://github.com/jayesh2810" target="_blank" rel="noopener" class="hero__link">github</a>
			<span class="hero__sep">·</span>
			<a href="https://www.linkedin.com/in/jayesh-bhadane/" target="_blank" rel="noopener" class="hero__link">linkedin</a>
		</div>
	</div>
</section>

<style>
	.hero {
		position: relative;
		min-height: 100vh;
		display: flex;
		align-items: center;
		padding: 0 1.5rem;
		overflow: hidden;
	}

	.hero__content {
		position: relative;
		z-index: 1;
		max-width: 1100px;
		margin: 0 auto;
		width: 100%;
	}

	.hero__label,
	.hero__name,
	.hero__subtitle,
	.hero__links {
		opacity: 0;
		transform: translateY(24px);
		transition: opacity 0.6s var(--ease-smooth), transform 0.6s var(--ease-smooth);
	}

	.hero__content--visible .hero__label {
		opacity: 1; transform: translateY(0);
		transition-delay: 0.1s;
	}
	.hero__content--visible .hero__name {
		opacity: 1; transform: translateY(0);
		transition-delay: 0.25s;
	}
	.hero__content--visible .hero__subtitle {
		opacity: 1; transform: translateY(0);
		transition-delay: 0.45s;
	}
	.hero__content--visible .hero__links {
		opacity: 1; transform: translateY(0);
		transition-delay: 0.6s;
	}

	.hero__label {
		font-family: var(--font-mono);
		font-size: 0.85rem;
		color: var(--color-muted);
		display: block;
		margin-bottom: 1rem;
	}

	.hero__name {
		font-family: var(--font-serif);
		font-weight: 700;
		font-size: clamp(4.5rem, 10vw, 7.5rem);
		line-height: 1.05;
		color: var(--color-text);
		margin: 0 0 1.5rem;
	}

	.hero__dot {
		color: var(--color-accent);
	}

	.hero__subtitle {
		font-family: var(--font-sans);
		font-size: 1.2rem;
		line-height: 1.7;
		color: var(--color-muted);
		max-width: 560px;
		margin-bottom: 2rem;
	}

	.hero__links {
		display: flex;
		align-items: center;
		gap: 0.75rem;
	}

	.hero__link {
		font-family: var(--font-mono);
		font-size: 0.85rem;
		color: var(--color-muted);
		text-decoration: none;
		transition: color 0.2s var(--ease-smooth);
	}

	.hero__link:hover {
		color: var(--color-accent);
	}

	.hero__sep {
		color: var(--color-border);
		font-size: 1rem;
	}

	@media (max-width: 640px) {
		.hero__subtitle {
			font-size: 1.05rem;
		}
	}
</style>
