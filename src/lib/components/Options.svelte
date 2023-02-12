<script lang="ts">
	import { createEventDispatcher } from 'svelte';
	import type { Difficulty } from './Game.svelte';

	export let difficulties: Difficulty[];
	export let difficulty: number;

	const dispatch = createEventDispatcher<{ start: undefined }>();

	function start() {
		dispatch('start');
	}

	function setDifficulty(target: EventTarget | null) {
		if (target) {
			difficulty = parseInt((target as HTMLOptionElement).value);
		}
	}
</script>

<div class="options">
	<select lang="ts" bind:value={difficulty} on:change={({ target }) => setDifficulty(target)}>
		{#each difficulties as { name }, i}
			<option value={i}>{name}</option>
		{/each}
	</select>
	<button on:click={start}>New Game</button>
</div>

<style lang="scss">
	.options {
		display: flex;
		justify-content: center;
	}
</style>
