<script lang="ts">
	import { onMount } from 'svelte';

	let containerEl: HTMLElement;

	const projects = [
		{
			title: 'Travelers Insurance — Kaggle Competition Win',
			description: "Led the team to first place in Travelers' predictive modeling competition on Kaggle. Analyzed historical claims data to build a segmented rating plan for auto insurance risk.",
			tags: ['Kaggle', 'Classification', 'Risk Modeling', 'Python', 'Feature Engineering'],
			link: '',
			github: ''
		},
		{
			title: 'Object Detection using Hausdorff Distance',
			description: 'Published research on a shape-matching approach for object recognition using Hausdorff distance and shape context descriptors. Tested on MNIST and COIL datasets with a 0.72% error rate.',
			tags: ['Computer Vision', 'Shape Matching', 'MNIST', 'Published Paper', 'IRJET 2020'],
			link: 'https://www.irjet.net/archives/V7/i4/IRJET-V7I416.pdf',
			github: ''
		},
		{
			title: 'DeepImageSearch — Autoencoder + KNN',
			description: 'Content-based image retrieval system using a convolutional autoencoder to learn compressed visual features and K-nearest neighbors to find the most similar images to a query.',
			tags: ['Deep Learning', 'Autoencoder', 'KNN', 'TensorFlow', 'Computer Vision'],
			link: '',
			github: 'https://github.com/jayesh2810/DeepImageSearch-Autoencoder-KNN'
		},
		{
			title: 'AI-Powered HR Portal',
			description: 'Full-stack HR management system with AI-generated offer letters via CrewAI/OpenAI, PDF generation, Gmail sending, employee management in SQLite, and natural-language offer querying.',
			tags: ['CrewAI', 'OpenAI', 'Streamlit', 'SQLite', 'Flask', 'Docker'],
			link: '',
			github: 'https://github.com/jayesh2810/Full_stack'
		},
		{
			title: 'SarcasticAstroBot',
			description: 'Fine-tuned GPT-4o-mini to act as a sarcastic astronomy-obsessed physics professor, using custom JSONL training data and Weights & Biases for experiment tracking.',
			tags: ['Fine-Tuning', 'OpenAI', 'GPT-4o-mini', 'NLP', 'Weights & Biases'],
			link: '',
			github: 'https://github.com/jayesh2810/SarcasticAstroBot'
		},
		{
			title: 'YouTube-to-AI-Article Generator',
			description: 'Transcribes YouTube videos with OpenAI Whisper, rewrites the content into a polished article using GPT-4 and LangChain, and generates a DALL·E 3 cover image.',
			tags: ['Whisper', 'GPT-4', 'LangChain', 'DALL·E 3', 'Streamlit'],
			link: '',
			github: 'https://github.com/jayesh2810/YouTube-to-AI-Article-Generator'
		},
		{
			title: 'Binarization for OCR',
			description: 'Interactive Streamlit app to compare five binarization methods — Otsu, adaptive threshold, Niblack, Sauvola, and Bernsen — for OCR preprocessing, with side-by-side visualizations.',
			tags: ['OpenCV', 'Image Processing', 'OCR', 'Streamlit', 'Computer Vision'],
			link: '',
			github: 'https://github.com/jayesh2810/Binarization-for-OCR'
		}
	];

	let visibleItems = $state<boolean[]>(new Array(projects.length).fill(false));

	onMount(() => {
		const cards = containerEl.querySelectorAll('.project');
		const observer = new IntersectionObserver(
			(entries) => {
				for (const entry of entries) {
					if (entry.isIntersecting) {
						const idx = Number((entry.target as HTMLElement).dataset.idx);
						visibleItems[idx] = true;
					}
				}
			},
			{ threshold: 0.15 }
		);
		cards.forEach((card) => observer.observe(card));
		return () => observer.disconnect();
	});
</script>

<div class="projects" bind:this={containerEl}>
	{#each projects as project, i}
		<div
			class="project"
			class:project--visible={visibleItems[i]}
			data-idx={i}
		>
			<div class="project__accent"></div>
			<div class="project__info">
				<h3 class="project__title">{project.title}</h3>
				<p class="project__desc">{project.description}</p>
				<div class="project__tags">
					{#each project.tags as tag}
						<span class="project__tag">{tag}</span>
					{/each}
				</div>
				{#if project.link}
					<a href={project.link} target="_blank" rel="noopener" class="project__link">View paper ↗</a>
				{/if}
				{#if project.github}
					<a href={project.github} target="_blank" rel="noopener" class="project__link">GitHub ↗</a>
				{/if}
			</div>
		</div>
	{/each}
</div>

<style>
	.projects {
		display: flex;
		flex-direction: column;
		gap: 2rem;
	}

	.project {
		display: flex;
		background: var(--color-card);
		border: 1px solid var(--color-border);
		border-radius: 8px;
		overflow: hidden;
		opacity: 0;
		transform: translateY(28px);
		transition: opacity 0.55s var(--ease-smooth), transform 0.55s var(--ease-smooth),
			box-shadow 0.4s var(--ease-smooth), border-color 0.4s var(--ease-smooth);
	}

	.project--visible {
		opacity: 1;
		transform: translateY(0);
	}

	.project:hover {
		box-shadow: 0 8px 24px rgba(0, 0, 0, 0.05);
		border-color: var(--color-accent-light);
	}

	.project__accent {
		width: 4px;
		flex-shrink: 0;
		background: var(--color-accent);
	}

	.project__info {
		padding: 1.75rem 2rem;
	}

	.project__title {
		font-family: var(--font-serif);
		font-size: 1.3rem;
		font-weight: 700;
		color: var(--color-text);
		margin: 0 0 0.6rem;
		line-height: 1.3;
	}

	.project__desc {
		font-size: 0.95rem;
		line-height: 1.65;
		color: var(--color-muted);
		margin: 0 0 1rem;
	}

	.project__tags {
		display: flex;
		flex-wrap: wrap;
		gap: 0.4rem;
	}

	.project__tag {
		font-family: var(--font-mono);
		font-size: 0.7rem;
		color: var(--color-muted);
		background: var(--color-tag);
		padding: 0.25rem 0.6rem;
		border-radius: 4px;
	}

	.project__link {
		display: inline-block;
		margin-top: 1rem;
		font-family: var(--font-mono);
		font-size: 0.8rem;
		color: var(--color-accent);
		text-decoration: none;
		transition: opacity 0.2s var(--ease-smooth);
	}

	.project__link:hover {
		opacity: 0.7;
	}

	@media (max-width: 640px) {
		.project__info {
			padding: 1.25rem 1.25rem;
		}
	}
</style>
