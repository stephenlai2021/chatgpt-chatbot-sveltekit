<script lang="ts">
	import type { ChatCompletionRequestMessage } from 'openai'
	import { SSE } from 'sse.js'
	import { cubicOut } from 'svelte/easing'
	import { menuOpen } from '$lib/stores'
	import ChatMessage from '$lib/components/ChatMessage.svelte'

	let query: string = ''
	let answer: string = ''
	let apiKey: string = ''
	let loading: boolean = false
	let chatMessages: ChatCompletionRequestMessage[] = []
	let scrollToDiv: HTMLDivElement

	function scrollToBottom() {
		setTimeout(function () {
			scrollToDiv.scrollIntoView({ behavior: 'smooth', block: 'end', inline: 'nearest' })
		}, 100)
	}

	const handleSubmit = async () => {
		loading = true
		chatMessages = [...chatMessages, { role: 'user', content: query }]

		const eventSource = new SSE('/api/chat', {
			headers: {
				'Content-Type': 'application/json'
			},
			payload: JSON.stringify({ messages: chatMessages })
		})

		query = ''

		eventSource.addEventListener('error', handleError)

		eventSource.addEventListener('message', (e) => {
			scrollToBottom()
			try {
				loading = false
				if (e.data === '[DONE]') {
					chatMessages = [...chatMessages, { role: 'assistant', content: answer }]
					answer = ''
					return
				}

				const completionResponse = JSON.parse(e.data)
				const [{ delta }] = completionResponse.choices

				if (delta.content) {
					answer = (answer ?? '') + delta.content
				}
			} catch (err) {
				handleError(err)
			}
		})
		eventSource.stream()
		scrollToBottom()
	}

	function handleError<T>(err: T) {
		loading = false
		query = ''
		answer = ''
		console.error(err)
	}

	function slidefade(node, params) {
		const existingTransform = getComputedStyle(node).transform.replace('none', '')

		return {
			delay: params.delay || 0,
			duration: params.duration || 400,
			easing: params.easing || cubicOut,
			css: (t, u) =>
				`transform-origin: bottom right; transform: ${existingTransform} scaleY(${t}) scaleX(${t}); opacity: ${t};`
		}
	}
</script>

<div
	transition:slidefade
	class="relative pt-[10px] flex flex-col pt-4 w- px-0 items-center gap-"
	on:click|stopPropagation
	on:keydown
>	
	<div class="absolute h-[20px] w-full rounded-[8px] z-10 backdrop-blur-[6px]" />	
	<div class="h-[300px] w-[350px] pt-[0px] bg-gray-900 rounded-t-[8px] p-4 overflow-y-auto flex flex-col gap-4">
		<div class="flex flex-col gap-2 pt-[16px]">
			<ChatMessage type="assistant" message="Hello, ask me anything you want!" />
			{#each chatMessages as message}
				<ChatMessage type={message.role} message={message.content} />
			{/each}
			{#if answer}
				<ChatMessage type="assistant" message={answer} />
			{/if}3
			{#if loading}
				<ChatMessage type="assistant" message="Loading.." />
			{/if}
		</div>
		<div class="" bind:this={scrollToDiv} />
	</div>
	<form
		class="relative flex w-full border border-t-gray-600 rounded-b-[8px] gap-4 bg-gray-900 p-4"
		on:submit|preventDefault={handleSubmit}
	>		
		<input type="text" class="rounded-[4px] py-[3px] pl-[5px] bordered w-full" bind:value={query} />
		<button type="submit" class="rounded-[4px] px-[10px] btn-accent">Send</button>
	</form>
</div>