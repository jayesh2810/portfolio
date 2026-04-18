<script lang="ts">
	const skills = [
		'Python', 'PyTorch', 'Scikit-learn', 'SQL', 'R', 'FastAPI', 'Docker',
		'Svelte', 'LLMs', 'RAG', 'Multi-Agent Systems', 'Bayesian Methods',
		'A/B Testing', 'Time Series', 'NLP', 'Clustering', 'Deep Learning', 'Computer Vision'
	];

	let paused = $state(false);
</script>

<div
	class="ticker"
	onmouseenter={() => (paused = true)}
	onmouseleave={() => (paused = false)}
	role="marquee"
	aria-label="Skills"
>
	<div class="ticker__track" class:ticker__track--paused={paused}>
		{#each [0, 1] as _copy}
			<div class="ticker__group">
				{#each skills as skill}
					<span class="ticker__item">{skill}</span>
					<span class="ticker__sep" aria-hidden="true">·</span>
				{/each}
			</div>
		{/each}
	</div>
</div>

<style>
	.ticker {
		overflow: hidden;
		padding: 3rem 0;
		border-top: 1px solid var(--color-border);
		border-bottom: 1px solid var(--color-border);
		margin: 5rem 0 0;
	}

	.ticker__track {
		display: flex;
		width: max-content;
		animation: scroll 40s linear infinite;
	}

	.ticker__track--paused {
		animation-play-state: paused;
	}

	.ticker__group {
		display: flex;
		align-items: center;
		flex-shrink: 0;
	}

	.ticker__item {
		font-family: var(--font-mono);
		font-size: 0.85rem;
		color: var(--color-muted);
		white-space: nowrap;
	}

	.ticker__sep {
		margin: 0 1rem;
		color: var(--color-border);
		font-size: 1.1rem;
	}

	@keyframes scroll {
		from { transform: translateX(0); }
		to { transform: translateX(-50%); }
	}
</style>
