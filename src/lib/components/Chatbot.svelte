<script lang="ts">
	import { onMount } from 'svelte';

	interface Message {
		role: 'user' | 'assistant';
		content: string;
	}

	let open = $state(false);
	let input = $state('');
	let messages = $state<Message[]>([]);
	let streaming = $state(false);
	let userMsgCount = $state(0);
	let limitReached = $state(false);
	let errorMsg = $state('');
	let messagesEl = $state<HTMLElement>(null!);
	let inputEl = $state<HTMLTextAreaElement>(null!);
	let pulsed = $state(false);

	const MAX_CHARS = 500;
	const MAX_USER_MSGS = 10;

	const suggestions = [
		'What has he built recently?',
		"What's his strongest skill area?",
		'Is he open to contract roles?'
	];

	const welcomeMessage: Message = {
		role: 'assistant',
		content:
			"Hey! I know everything on Jayesh's resume. Ask me anything — his experience, skills, projects, or what he's looking for."
	};

	onMount(() => {
		messages = [welcomeMessage];
		setTimeout(() => { pulsed = true; }, 2000);
	});

	function scrollToBottom() {
		requestAnimationFrame(() => {
			if (messagesEl) messagesEl.scrollTop = messagesEl.scrollHeight;
		});
	}

	function toggleOpen() {
		open = !open;
		if (open) {
			requestAnimationFrame(() => inputEl?.focus());
			scrollToBottom();
		}
	}

	async function sendMessage(text: string) {
		if (!text.trim() || streaming || limitReached) return;

		errorMsg = '';
		const userMsg: Message = { role: 'user', content: text.trim() };
		messages = [...messages, userMsg];
		userMsgCount++;
		input = '';
		scrollToBottom();

		if (userMsgCount >= MAX_USER_MSGS) {
			limitReached = true;
		}

		streaming = true;
		const assistantMsg: Message = { role: 'assistant', content: '' };
		messages = [...messages, assistantMsg];
		scrollToBottom();

		// Build history: skip the welcome message (index 0), drop the empty assistant
		// placeholder we just pushed, and any prior error bubbles. The filter on
		// `m.content` already removes the empty placeholder, so do NOT slice it again
		// (that would chop off the user's latest question and send an empty turn).
		const conversationHistory = messages
			.slice(1)
			.filter((m) => m.content && !isErrorBubble(m.content))
			.map((m) => ({ role: m.role, content: m.content }));

		const NETWORK_ERROR_MSG =
			"I'm having trouble reaching the AI service right now. Please try again in a moment, or reach out to Jayesh directly at jayesh281998@gmail.com.";
		const EMPTY_COMPLETION_MSG =
			"I couldn't generate a good answer for that. Try rephrasing, or ask me something else about Jayesh's experience, skills, or projects.";

		let serverSignaledError: 'empty_completion' | 'stream_error' | null = null;

		try {
			const res = await fetch('/api/chat', {
				method: 'POST',
				headers: { 'Content-Type': 'application/json' },
				body: JSON.stringify({ messages: conversationHistory })
			});

			if (!res.ok) {
				const err = await res.json().catch(() => null);
				throw new Error(err?.error || NETWORK_ERROR_MSG);
			}

			const reader = res.body!.getReader();
			const decoder = new TextDecoder();
			let buffer = '';

			outer: while (true) {
				const { done, value } = await reader.read();
				if (done) break;

				buffer += decoder.decode(value, { stream: true });
				const lines = buffer.split('\n');
				buffer = lines.pop() || '';

				for (const line of lines) {
					const trimmed = line.trim();
					if (!trimmed || !trimmed.startsWith('data: ')) continue;
					const data = trimmed.slice(6);
					if (data === '[DONE]') break outer;

					try {
						const parsed = JSON.parse(data);
						if (parsed.error) {
							serverSignaledError = parsed.error;
							continue;
						}
						if (parsed.content) {
							assistantMsg.content += parsed.content;
							messages = [...messages.slice(0, -1), { ...assistantMsg }];
							scrollToBottom();
						}
					} catch {
						continue;
					}
				}
			}

			if (!assistantMsg.content) {
				assistantMsg.content =
					serverSignaledError === 'empty_completion'
						? EMPTY_COMPLETION_MSG
						: NETWORK_ERROR_MSG;
				messages = [...messages.slice(0, -1), { ...assistantMsg }];
			}
		} catch (e) {
			assistantMsg.content = e instanceof Error ? e.message : NETWORK_ERROR_MSG;
			messages = [...messages.slice(0, -1), { ...assistantMsg }];
		} finally {
			streaming = false;
			scrollToBottom();
		}
	}

	function isErrorBubble(content: string): boolean {
		return (
			content.startsWith('Something went wrong') ||
			content.startsWith("I'm having trouble reaching") ||
			content.startsWith("I couldn't generate")
		);
	}

	function handleKeydown(e: KeyboardEvent) {
		if (e.key === 'Enter' && !e.shiftKey) {
			e.preventDefault();
			sendMessage(input);
		}
	}

	function handleSuggestion(text: string) {
		sendMessage(text);
	}
</script>

<!-- Floating button -->
<button
	class="chat-fab"
	class:chat-fab--pulsed={pulsed}
	onclick={toggleOpen}
	aria-label={open ? 'Close chat' : 'Open chat'}
>
	{#if open}
		<svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
			<line x1="18" y1="6" x2="6" y2="18"/><line x1="6" y1="6" x2="18" y2="18"/>
		</svg>
	{:else}
		<svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
			<path d="M21 15a2 2 0 0 1-2 2H7l-4 4V5a2 2 0 0 1 2-2h14a2 2 0 0 1 2 2z"/>
		</svg>
	{/if}
</button>

<!-- Chat panel -->
{#if open}
	<div class="chat-panel">
		<div class="chat-header">
			<span class="chat-header__title">Ask about Jayesh</span>
			<button class="chat-header__close" onclick={toggleOpen} aria-label="Close chat">×</button>
		</div>

		<div class="chat-messages" bind:this={messagesEl}>
			{#each messages as msg, i}
				<div class="chat-msg" class:chat-msg--user={msg.role === 'user'} class:chat-msg--assistant={msg.role === 'assistant'}>
					<div class="chat-bubble" class:chat-bubble--user={msg.role === 'user'} class:chat-bubble--assistant={msg.role === 'assistant'}>
						{msg.content}
					</div>
				</div>

				<!-- Suggestions after the first welcome message only -->
				{#if i === 0 && msg.role === 'assistant' && messages.length === 1}
					<div class="chat-suggestions">
						{#each suggestions as s}
							<button class="chat-chip" onclick={() => handleSuggestion(s)}>{s}</button>
						{/each}
					</div>
				{/if}
			{/each}

			{#if streaming && messages[messages.length - 1]?.content === ''}
				<div class="chat-msg chat-msg--assistant">
					<div class="chat-bubble chat-bubble--assistant chat-typing">
						<span class="dot"></span><span class="dot"></span><span class="dot"></span>
					</div>
				</div>
			{/if}
		</div>

		<div class="chat-input-area">
			{#if limitReached}
				<div class="chat-limit">
					That's the limit for this widget! For a deeper conversation, reach out to Jayesh directly at
					<a href="mailto:jayesh281998@gmail.com">jayesh281998@gmail.com</a>.
				</div>
			{:else}
				<div class="chat-input-wrap">
					<textarea
						bind:this={inputEl}
						bind:value={input}
						class="chat-input"
						placeholder="Ask something..."
						maxlength={MAX_CHARS}
						rows="1"
						onkeydown={handleKeydown}
						disabled={streaming}
					></textarea>
					<span class="chat-charcount" class:chat-charcount--warn={input.length > MAX_CHARS - 50}>
						{input.length}/{MAX_CHARS}
					</span>
					<button
						class="chat-send"
						onclick={() => sendMessage(input)}
						disabled={!input.trim() || streaming}
						aria-label="Send message"
					>
						<svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
							<line x1="12" y1="19" x2="12" y2="5"/><polyline points="5 12 12 5 19 12"/>
						</svg>
					</button>
				</div>
			{/if}
		</div>
	</div>
{/if}

<style>
	/* --- Floating action button --- */
	.chat-fab {
		position: fixed;
		bottom: 20px;
		right: 20px;
		width: 56px;
		height: 56px;
		border-radius: 50%;
		background: var(--color-accent);
		color: #fff;
		border: none;
		cursor: pointer;
		display: flex;
		align-items: center;
		justify-content: center;
		box-shadow: 0 4px 16px rgba(196, 93, 62, 0.35);
		z-index: 200;
		transition: transform 0.2s var(--ease-smooth), box-shadow 0.2s var(--ease-smooth);
	}

	.chat-fab:hover {
		transform: scale(1.08);
		box-shadow: 0 6px 24px rgba(196, 93, 62, 0.45);
	}

	.chat-fab--pulsed {
		animation: fab-pulse 1.2s ease-out 1;
	}

	@keyframes fab-pulse {
		0% { box-shadow: 0 4px 16px rgba(196, 93, 62, 0.35); }
		50% { box-shadow: 0 0 0 12px rgba(196, 93, 62, 0); }
		100% { box-shadow: 0 4px 16px rgba(196, 93, 62, 0.35); }
	}

	/* --- Chat panel --- */
	.chat-panel {
		position: fixed;
		bottom: 88px;
		right: 20px;
		width: 380px;
		height: 520px;
		background: #FFFFFF;
		border-radius: 12px 12px 0 12px;
		box-shadow: 0 8px 40px rgba(0, 0, 0, 0.12);
		z-index: 199;
		display: flex;
		flex-direction: column;
		animation: panel-up 0.3s ease-out;
		overflow: hidden;
	}

	@keyframes panel-up {
		from { opacity: 0; transform: translateY(16px) scale(0.97); }
		to { opacity: 1; transform: translateY(0) scale(1); }
	}

	/* --- Header --- */
	.chat-header {
		display: flex;
		align-items: center;
		justify-content: space-between;
		padding: 0.85rem 1rem;
		border-bottom: 1px solid var(--color-border);
		flex-shrink: 0;
	}

	.chat-header__title {
		font-family: var(--font-mono);
		font-size: 0.85rem;
		font-weight: 500;
		color: var(--color-text);
	}

	.chat-header__close {
		background: none;
		border: none;
		font-size: 1.4rem;
		color: var(--color-muted);
		cursor: pointer;
		line-height: 1;
		padding: 0 4px;
		transition: color 0.15s;
	}

	.chat-header__close:hover {
		color: var(--color-text);
	}

	/* --- Messages area --- */
	.chat-messages {
		flex: 1;
		overflow-y: auto;
		padding: 1rem;
		display: flex;
		flex-direction: column;
		gap: 0.6rem;
	}

	.chat-msg {
		display: flex;
	}

	.chat-msg--user {
		justify-content: flex-end;
	}

	.chat-msg--assistant {
		justify-content: flex-start;
	}

	.chat-bubble {
		max-width: 82%;
		padding: 0.6rem 0.85rem;
		border-radius: 12px;
		font-size: 0.87rem;
		line-height: 1.55;
		word-break: break-word;
		white-space: pre-wrap;
	}

	.chat-bubble--user {
		background: var(--color-accent);
		color: #fff;
		border-bottom-right-radius: 4px;
	}

	.chat-bubble--assistant {
		background: var(--color-bg-dark);
		color: var(--color-text);
		border-bottom-left-radius: 4px;
	}

	/* --- Suggestions --- */
	.chat-suggestions {
		display: flex;
		flex-wrap: wrap;
		gap: 0.4rem;
		padding: 0.25rem 0 0.5rem;
	}

	.chat-chip {
		font-family: var(--font-mono);
		font-size: 0.7rem;
		color: var(--color-accent);
		background: var(--color-tag);
		border: 1px solid var(--color-border);
		border-radius: 999px;
		padding: 0.3rem 0.7rem;
		cursor: pointer;
		transition: background 0.15s, border-color 0.15s;
	}

	.chat-chip:hover {
		background: var(--color-accent-light);
		border-color: var(--color-accent);
	}

	/* --- Typing indicator --- */
	.chat-typing {
		display: flex;
		align-items: center;
		gap: 4px;
		padding: 0.65rem 1rem;
	}

	.dot {
		width: 6px;
		height: 6px;
		background: var(--color-muted);
		border-radius: 50%;
		animation: dot-bounce 1.4s ease-in-out infinite;
	}

	.dot:nth-child(2) { animation-delay: 0.2s; }
	.dot:nth-child(3) { animation-delay: 0.4s; }

	@keyframes dot-bounce {
		0%, 80%, 100% { transform: scale(0.6); opacity: 0.4; }
		40% { transform: scale(1); opacity: 1; }
	}

	/* --- Input area --- */
	.chat-input-area {
		flex-shrink: 0;
		padding: 0.65rem 0.75rem;
		border-top: 1px solid var(--color-border);
	}

	.chat-input-wrap {
		display: flex;
		align-items: flex-end;
		gap: 0.5rem;
		position: relative;
	}

	.chat-input {
		flex: 1;
		font-family: var(--font-sans);
		font-size: 0.87rem;
		color: var(--color-text);
		background: transparent;
		border: none;
		border-bottom: 1.5px solid var(--color-border);
		padding: 0.45rem 0.25rem;
		resize: none;
		outline: none;
		transition: border-color 0.2s;
		max-height: 80px;
	}

	.chat-input:focus {
		border-bottom-color: var(--color-accent);
	}

	.chat-input::placeholder {
		color: var(--color-muted);
	}

	.chat-charcount {
		position: absolute;
		right: 42px;
		bottom: 6px;
		font-family: var(--font-mono);
		font-size: 0.6rem;
		color: var(--color-muted);
		pointer-events: none;
	}

	.chat-charcount--warn {
		color: var(--color-accent);
	}

	.chat-send {
		width: 32px;
		height: 32px;
		border-radius: 50%;
		background: var(--color-accent);
		color: #fff;
		border: none;
		cursor: pointer;
		display: flex;
		align-items: center;
		justify-content: center;
		flex-shrink: 0;
		transition: opacity 0.15s;
	}

	.chat-send:disabled {
		opacity: 0.4;
		cursor: not-allowed;
	}

	.chat-limit {
		font-family: var(--font-sans);
		font-size: 0.8rem;
		color: var(--color-muted);
		text-align: center;
		line-height: 1.5;
		padding: 0.5rem 0;
	}

	.chat-limit a {
		color: var(--color-accent);
		text-decoration: none;
	}

	/* --- Mobile --- */
	@media (max-width: 640px) {
		.chat-panel {
			width: 100%;
			height: 70vh;
			bottom: 0;
			right: 0;
			border-radius: 12px 12px 0 0;
		}

		.chat-fab {
			bottom: 20px;
			right: 20px;
		}
	}
</style>
