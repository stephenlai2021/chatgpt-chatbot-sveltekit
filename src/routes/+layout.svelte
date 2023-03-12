<script>
	import '../app.css'
	import { menuOpen } from '$lib/stores'
	import ChatWindow from '$lib/components/ChatWindow.svelte'
	import admin from '$lib/assets/sister.png'
</script>

<div class="layout">
	<slot />
		<img
			src={admin}
			alt=""
			width="50"
			class="icon-admin"
			on:click={() => ($menuOpen = !$menuOpen)}
		/>
	{#if $menuOpen}
		<div class="chat-window">
			<ChatWindow />
		</div>
	{/if}
</div>

<style>
	.icon-admin {
		position: fixed;
		bottom: 30px;
		right: 30px;
		cursor: pointer;
		border-radius: 50%;
	}

	.layout {
		overflow: hidden;
		height: 100vh;
	}

	.chat-window {
		position: fixed;
		bottom: 90px;
		right: 30px;
	}

	/* @media (max-width: 475px) {
		.container {
			width: 100%;
		}
	} */
</style>
