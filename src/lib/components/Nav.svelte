<script lang="ts">
	let scrolled = $state(false);
	let mobileOpen = $state(false);

	const links = [
		{ label: 'about', href: '#about' },
		{ label: 'work', href: '#work' },
		{ label: 'contact', href: '#contact' }
	];

	function handleScroll() {
		scrolled = window.scrollY > 50;
	}

	function closeMobile() {
		mobileOpen = false;
	}

	function handleNavClick(e: MouseEvent, href: string) {
		e.preventDefault();
		closeMobile();
		const el = document.querySelector(href);
		el?.scrollIntoView({ behavior: 'smooth' });
	}
</script>

<svelte:window onscroll={handleScroll} />

<nav class="nav" class:nav--scrolled={scrolled}>
	<div class="nav__inner">
		<!-- svelte-ignore a11y_invalid_attribute -->
		<a href="#home" class="nav__logo" onclick={(e) => { e.preventDefault(); window.scrollTo({ top: 0, behavior: 'smooth' }); }}>J.</a>

		<button class="nav__hamburger" onclick={() => (mobileOpen = !mobileOpen)} aria-label="Toggle menu">
			<span class="nav__hamburger-line" class:open={mobileOpen}></span>
			<span class="nav__hamburger-line" class:open={mobileOpen}></span>
			<span class="nav__hamburger-line" class:open={mobileOpen}></span>
		</button>

		<ul class="nav__links" class:nav__links--open={mobileOpen}>
			{#each links as link}
				<li>
					<a href={link.href} class="nav__link" onclick={(e) => handleNavClick(e, link.href)}>
						{link.label}
					</a>
				</li>
			{/each}
		</ul>
	</div>
</nav>

{#if mobileOpen}
	<button class="nav__overlay" onclick={closeMobile} aria-label="Close menu"></button>
{/if}

<style>
	.nav {
		position: fixed;
		top: 0;
		left: 0;
		right: 0;
		z-index: 100;
		transition: background-color 0.3s var(--ease-smooth), border-color 0.3s var(--ease-smooth);
		border-bottom: 1px solid transparent;
	}

	.nav--scrolled {
		background: rgba(250, 247, 242, 0.85);
		backdrop-filter: blur(12px);
		-webkit-backdrop-filter: blur(12px);
		border-bottom-color: var(--color-border);
	}

	.nav__inner {
		max-width: 1100px;
		margin: 0 auto;
		padding: 1.25rem 1.5rem;
		display: flex;
		align-items: center;
		justify-content: space-between;
	}

	.nav__logo {
		font-family: var(--font-serif);
		font-size: 1.75rem;
		font-weight: 700;
		color: var(--color-accent);
		text-decoration: none;
		line-height: 1;
	}

	.nav__links {
		display: flex;
		gap: 2rem;
		list-style: none;
		margin: 0;
		padding: 0;
	}

	.nav__link {
		font-family: var(--font-mono);
		font-size: 0.8rem;
		text-transform: lowercase;
		letter-spacing: 0.05em;
		color: var(--color-muted);
		text-decoration: none;
		transition: color 0.2s var(--ease-smooth);
	}

	.nav__link:hover {
		color: var(--color-accent);
	}

	.nav__hamburger {
		display: none;
		flex-direction: column;
		gap: 5px;
		background: none;
		border: none;
		cursor: pointer;
		padding: 4px;
	}

	.nav__hamburger-line {
		display: block;
		width: 24px;
		height: 2px;
		background: var(--color-text);
		transition: transform 0.3s var(--ease-smooth), opacity 0.3s var(--ease-smooth);
	}

	.nav__hamburger-line.open:nth-child(1) {
		transform: translateY(7px) rotate(45deg);
	}
	.nav__hamburger-line.open:nth-child(2) {
		opacity: 0;
	}
	.nav__hamburger-line.open:nth-child(3) {
		transform: translateY(-7px) rotate(-45deg);
	}

	.nav__overlay {
		display: none;
		position: fixed;
		inset: 0;
		background: rgba(0, 0, 0, 0.3);
		z-index: 90;
		border: none;
		cursor: default;
	}

	@media (max-width: 640px) {
		.nav__hamburger {
			display: flex;
		}

		.nav__links {
			position: fixed;
			top: 0;
			right: 0;
			bottom: 0;
			width: 260px;
			flex-direction: column;
			background: var(--color-bg);
			padding: 5rem 2rem 2rem;
			gap: 1.5rem;
			transform: translateX(100%);
			transition: transform 0.3s var(--ease-smooth);
			z-index: 95;
			box-shadow: -4px 0 24px rgba(0, 0, 0, 0.08);
		}

		.nav__links--open {
			transform: translateX(0);
		}

		.nav__link {
			font-size: 1rem;
		}

		.nav__overlay {
			display: block;
		}
	}
</style>
