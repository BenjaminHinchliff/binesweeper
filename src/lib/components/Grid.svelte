<script lang="ts">
	import { createEventDispatcher } from 'svelte';
	import { zip } from 'underscore';
	import { State, Mine } from './Game.svelte';

	export let counts: number[][];
	export let visible: State[][];
	export let alive: boolean;

	$: width = counts[0].length;

	const dispatch = createEventDispatcher<{
		reveal: { row: number; col: number };
		flag: { row: number; col: number };
	}>();

	function reveal(row: number, col: number) {
		if (alive) {
			dispatch('reveal', { row, col });
		}
	}

	function flag(row: number, col: number) {
		if (alive) {
			dispatch('flag', { row, col });
		}
	}
</script>

<div class="grid" style:grid-template-columns="repeat({width}, 1fr [col-start])">
	{#each zip(counts, visible) as [countRow, visRow], i}
		{#each zip(countRow, visRow) as [count, state], j}
			<button
				class="grid-cell"
				on:click={() => reveal(i, j)}
				on:contextmenu|preventDefault={() => flag(i, j)}
				disabled={state === State.Revealed}
				style:font-size="calc({0.55 / width} * var(--width))"
			>
				{state === State.Revealed
					? count === Mine
						? '💣'
						: count === 0
						? ' '
						: count
					: state === State.Flagged
					? '🚩'
					: ' '}
			</button>
		{/each}
	{/each}
</div>

<style lang="scss">
	.grid {
		display: grid;
		--width: min(100vw, 90vh);
		max-width: var(--width);
		margin: auto;
	}

	.grid-cell {
		font-size: calc(0.06 * var(--width));
		aspect-ratio: 1;
	}
</style>
