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
</script>

<footer id="contact" class="footer" bind:this={sectionEl}>
	<div class="footer__inner" class:footer__inner--visible={visible}>
		<h2 class="footer__heading">Let's talk.</h2>
		<p class="footer__subtitle">Currently open to Data Scientist and ML Engineer roles at early-stage startups.</p>

		<a href="mailto:jayesh281998@gmail.com" class="footer__email">jayesh281998@gmail.com</a>

		<div class="footer__socials">
			<a href="https://linkedin.com/in/jayeshbhadane" target="_blank" rel="noopener" aria-label="LinkedIn" class="footer__social">
				<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round" width="22" height="22">
					<path d="M16 8a6 6 0 0 1 6 6v7h-4v-7a2 2 0 0 0-4 0v7h-4v-7a6 6 0 0 1 6-6z"/>
					<rect x="2" y="9" width="4" height="12"/>
					<circle cx="4" cy="4" r="2"/>
				</svg>
			</a>
			<a href="https://github.com/jayesh2810" target="_blank" rel="noopener" aria-label="GitHub" class="footer__social">
				<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round" width="22" height="22">
					<path d="M9 19c-5 1.5-5-2.5-7-3m14 6v-3.87a3.37 3.37 0 0 0-.94-2.61c3.14-.35 6.44-1.54 6.44-7A5.44 5.44 0 0 0 20 4.77 5.07 5.07 0 0 0 19.91 1S18.73.65 16 2.48a13.38 13.38 0 0 0-7 0C6.27.65 5.09 1 5.09 1A5.07 5.07 0 0 0 5 4.77a5.44 5.44 0 0 0-1.5 3.78c0 5.42 3.3 6.61 6.44 7A3.37 3.37 0 0 0 9 18.13V22"/>
				</svg>
			</a>
			<a href="https://twitter.com/jayeshbhadane" target="_blank" rel="noopener" aria-label="Twitter" class="footer__social">
				<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round" width="22" height="22">
					<path d="M4 4l11.733 16h4.267l-11.733-16zM4 20l6.768-6.768M20 4l-6.768 6.768"/>
				</svg>
			</a>
		</div>

		<p class="footer__copy">© {new Date().getFullYear()} Jayesh Bhadane. All rights reserved.</p>
	</div>
</footer>

<style>
	.footer {
		background: var(--color-dark);
		padding: 8rem 1.5rem 3rem;
	}

	.footer__inner {
		max-width: 1100px;
		margin: 0 auto;
		text-align: center;
		opacity: 0;
		transform: translateY(24px);
		transition: opacity 0.6s var(--ease-smooth), transform 0.6s var(--ease-smooth);
	}

	.footer__inner--visible {
		opacity: 1;
		transform: translateY(0);
	}

	.footer__heading {
		font-family: var(--font-serif);
		font-size: clamp(2.5rem, 5vw, 4rem);
		font-weight: 700;
		color: #FFFFFF;
		margin: 0 0 1rem;
	}

	.footer__subtitle {
		font-size: 1.05rem;
		color: rgba(255, 255, 255, 0.55);
		margin: 0 0 2.5rem;
		line-height: 1.6;
	}

	.footer__email {
		font-family: var(--font-mono);
		font-size: clamp(1rem, 2.5vw, 1.35rem);
		color: var(--color-accent);
		text-decoration: none;
		display: inline-block;
		margin-bottom: 2.5rem;
		transition: opacity 0.2s var(--ease-smooth);
	}

	.footer__email:hover {
		opacity: 0.75;
	}

	.footer__socials {
		display: flex;
		justify-content: center;
		gap: 1.5rem;
		margin-bottom: 4rem;
	}

	.footer__social {
		color: rgba(255, 255, 255, 0.45);
		transition: color 0.2s var(--ease-smooth);
	}

	.footer__social:hover {
		color: var(--color-accent);
	}

	.footer__copy {
		font-family: var(--font-mono);
		font-size: 0.7rem;
		color: rgba(255, 255, 255, 0.25);
	}
</style>
