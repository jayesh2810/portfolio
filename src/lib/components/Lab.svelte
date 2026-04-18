<script lang="ts">
	import { onMount } from 'svelte';
	import ClusteringPlayground from './ClusteringPlayground.svelte';

	let sectionEl: HTMLElement;
	let visible = $state(false);

	onMount(() => {
		const observer = new IntersectionObserver(
			([entry]) => { if (entry.isIntersecting) visible = true; },
			{ threshold: 0.1 }
		);
		observer.observe(sectionEl);
		return () => observer.disconnect();
	});
</script>

<section id="lab" class="lab" bind:this={sectionEl}>
	<div class="lab__inner" class:lab__inner--visible={visible}>
		<span class="lab__label">{'// interactive experiments'}</span>
		<h2 class="lab__heading">Lab</h2>

		<ClusteringPlayground />
	</div>
</section>

<style>
	.lab {
		background: var(--color-bg-dark);
		padding: 8rem 1.5rem;
	}

	.lab__inner {
		max-width: 1100px;
		margin: 0 auto;
		opacity: 0;
		transform: translateY(32px);
		transition: opacity 0.6s var(--ease-smooth), transform 0.6s var(--ease-smooth);
	}

	.lab__inner--visible {
		opacity: 1;
		transform: translateY(0);
	}

	.lab__label {
		font-family: var(--font-mono);
		font-size: 0.8rem;
		color: var(--color-muted);
		display: block;
		margin-bottom: 0.5rem;
	}

	.lab__heading {
		font-family: var(--font-serif);
		font-size: 2.5rem;
		font-weight: 700;
		color: var(--color-text);
		margin: 0 0 3rem;
	}
</style>
