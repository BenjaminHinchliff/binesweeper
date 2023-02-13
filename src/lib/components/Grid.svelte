<script lang="ts">
	import { createEventDispatcher } from 'svelte';
	import { zip } from 'underscore';
	import { State, Mine } from './Game.svelte';
	import mineImg from '$lib/assets/mine-plain.svg';
	import flagImg from '$lib/assets/flag-plain.svg';

	export let counts: number[][];
	export let visible: State[][];
	export let alive: boolean;

	// TODO: tweak colors to be less... awful
	const colors = [
		'#0100fe',
		'#017f01',
		'#fe0000',
		'#010080',
		'#810102',
		'#008081',
		'#000000',
		'#808080'
	];

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
				style:color={colors[count - 1]}
			>
				{#if state === State.Revealed}
					{#if count === Mine}
						<img alt="mine" src={mineImg} />
					{:else if count !== 0}
						{count}
					{/if}
				{:else if state === State.Flagged}
					<img alt="flag" src={flagImg} />
				{/if}
			</button>
		{/each}
	{/each}
</div>

<style lang="scss">
	.grid {
		display: grid;
		column-gap: 0;
	}

	.grid-cell {
		font-size: calc(0.06 * var(--width));
		aspect-ratio: 1;
		margin: 0;
		padding: 0;
		border: 1px solid var(--form-element-border-color);

		&:disabled {
			opacity: 0.8;
		}

		&:active,
		&:focus {
			--background-color: var(--primary);
		}
	}

	img {
		width: 80%;
		height: 80%;
		margin: 10%;
	}
</style>
