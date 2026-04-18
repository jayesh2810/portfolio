<script lang="ts">
	import { onMount } from 'svelte';

	let sectionEl: HTMLElement;
	let visible = $state(false);

	onMount(() => {
		const observer = new IntersectionObserver(
			([entry]) => { if (entry.isIntersecting) visible = true; },
			{ threshold: 0.15 }
		);
		observer.observe(sectionEl);
		return () => observer.disconnect();
	});

	const stats = [
		'5 years experience',
		'3 industries',
		'MS Data Science, UConn'
	];
</script>

<section id="about" class="about" bind:this={sectionEl}>
	<div class="about__inner" class:about__inner--visible={visible}>
		<div class="about__text">
			<p>
				I'm a data scientist and ML engineer based in the Bay Area. I've spent the last five years working across enterprise analytics, deep learning research, and production AI systems — always at the intersection of "does this actually work?" and "does this actually matter?"
			</p>
			<p>
				At C5i, I worked with Fortune 500 teams on customer segmentation, marketing mix modeling, and A/B testing — the kind of work where a well-framed question matters more than a complex model. At Aurora Engineering, I shifted to spaceflight telemetry for NASA's MMS mission, building pipelines to recover missing data from satellite instruments. Most recently at Skan AI, I've been building agentic AI systems, LLM pipelines, and multi-agent architectures.
			</p>
			<p>
				I have a Master's in Data Science from UConn. What I care about most is solving problems that sit at the intersection of data, product thinking, and decision-making — and shipping solutions that hold up in the real world.
			</p>
		</div>

		<div class="about__side">
			<img src="/snap.jpeg" alt="Jayesh Bhadane" class="about__photo" />
			<div class="about__stats">
				{#each stats as stat}
					<span class="about__stat">{stat}</span>
				{/each}
			</div>
		</div>
	</div>
</section>

<style>
	.about {
		padding: 10rem 1.5rem 8rem;
	}

	.about__inner {
		max-width: 1100px;
		margin: 0 auto;
		display: grid;
		grid-template-columns: 1fr 0.65fr;
		gap: 4rem;
		align-items: start;
		opacity: 0;
		transform: translateY(32px);
		transition: opacity 0.6s var(--ease-smooth), transform 0.6s var(--ease-smooth);
	}

	.about__inner--visible {
		opacity: 1;
		transform: translateY(0);
	}

	.about__text {
		font-size: 1.05rem;
		line-height: 1.8;
		color: var(--color-text);
	}

	.about__text p {
		margin: 0 0 1.5rem;
	}

	.about__text p:last-child {
		margin-bottom: 0;
	}

	.about__side {
		position: sticky;
		top: 8rem;
	}

	.about__photo {
		width: 100%;
		aspect-ratio: 3 / 4;
		border-radius: 8px;
		object-fit: cover;
		margin-bottom: 1.5rem;
	}

	.about__stats {
		display: flex;
		flex-wrap: wrap;
		gap: 0.5rem;
	}

	.about__stat {
		font-family: var(--font-mono);
		font-size: 0.75rem;
		color: var(--color-muted);
		background: var(--color-accent-light);
		padding: 0.35rem 0.75rem;
		border-radius: 999px;
	}

	@media (max-width: 1024px) {
		.about__inner {
			grid-template-columns: 1fr;
			gap: 3rem;
		}

		.about__side {
			position: static;
			max-width: 320px;
		}
	}
</style>
