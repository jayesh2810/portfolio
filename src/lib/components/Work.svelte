<script lang="ts">
	import { onMount } from 'svelte';
	import Timeline from './Timeline.svelte';
	import Projects from './Projects.svelte';
	import SkillsTicker from './SkillsTicker.svelte';

	let sectionEl: HTMLElement;
	let visible = $state(false);

	onMount(() => {
		const observer = new IntersectionObserver(
			([entry]) => { if (entry.isIntersecting) visible = true; },
			{ threshold: 0.05 }
		);
		observer.observe(sectionEl);
		return () => observer.disconnect();
	});
</script>

<section id="work" class="work" bind:this={sectionEl}>
	<div class="work__inner">
		<div class="work__header" class:work__header--visible={visible}>
			<span class="work__label">{'// experience & projects'}</span>
			<h2 class="work__heading">Selected Work</h2>
		</div>

		<Timeline />
		<Projects />
		<SkillsTicker />
	</div>
</section>

<style>
	.work {
		padding: 8rem 1.5rem 2rem;
	}

	.work__inner {
		max-width: 1100px;
		margin: 0 auto;
	}

	.work__header {
		margin-bottom: 4rem;
		opacity: 0;
		transform: translateY(24px);
		transition: opacity 0.6s var(--ease-smooth), transform 0.6s var(--ease-smooth);
	}

	.work__header--visible {
		opacity: 1;
		transform: translateY(0);
	}

	.work__label {
		font-family: var(--font-mono);
		font-size: 0.8rem;
		color: var(--color-muted);
		display: block;
		margin-bottom: 0.5rem;
	}

	.work__heading {
		font-family: var(--font-serif);
		font-size: 2.5rem;
		font-weight: 700;
		color: var(--color-text);
		margin: 0;
	}
</style>
