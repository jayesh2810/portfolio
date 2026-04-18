<script lang="ts">
	import '../app.css';
	import Nav from '$lib/components/Nav.svelte';
	import Chatbot from '$lib/components/Chatbot.svelte';
	let { children } = $props();
</script>

<Nav />
<main>
	{@render children()}
</main>
<Chatbot />
