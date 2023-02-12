<script context="module" lang="ts">
	export enum Visiblity {
		Revealed,
		Flagged,
		Closed
	}
</script>

<script lang="ts">
	import { random } from 'underscore';

	import Grid from './Grid.svelte';

	export let width: number;
	export let height: number;
	export let mines: number;

	let counts: number[][] = Array.from({ length: height }, (_) => Array(width).fill(0));
	let visible: Visiblity[][] = Array.from({ length: height }, (_) => Array(width).fill(false));

	function forEachNeighbor(row: number, col: number, func: (i: number, j: number) => void) {
		for (let i = Math.max(row - 1, 0); i <= Math.min(row + 1, width - 1); i++) {
			for (let j = Math.max(col - 1, 0); j <= Math.min(col + 1, height - 1); j++) {
				if (i !== row || j !== col) {
					func(i, j);
				}
			}
		}
	}

	function incrementNeighbors(row: number, col: number) {
		forEachNeighbor(row, col, (i, j) => (counts[i][j] += 1));
	}

	function placeMines(counts: number[][]) {
		for (let n = 0; n < mines; n++) {
			while (true) {
				const i = random(0, counts.length - 1);
				const j = random(0, counts[0].length - 1);
				if (counts[i][j] !== Infinity) {
					counts[i][j] = Infinity;
					incrementNeighbors(i, j);
					break;
				}
			}
		}
	}

	function revealZeros(row: number, col: number) {
		visible[row][col] = Visiblity.Revealed;

		if (counts[row][col] != 0) {
			return;
		}

		// reveal non-zero neighbors
		forEachNeighbor(row, col, (i, j) => {
			if (visible[i][j] !== Visiblity.Revealed) {
				revealZeros(i, j);
			}
		});
	}

	placeMines(counts);

	function reveal(event: CustomEvent<{ row: number; col: number }>) {
		const { row, col } = event.detail;
		revealZeros(row, col);
		visible[row][col] = Visiblity.Revealed;
	}

	function flag(event: CustomEvent<{ row: number; col: number }>) {
		const { row, col } = event.detail;
		if (visible[row][col] === Visiblity.Flagged) {
			visible[row][col] = Visiblity.Closed;
		} else {
			visible[row][col] = Visiblity.Flagged;
		}
	}
</script>

<Grid {counts} {visible} on:reveal={reveal} on:flag={flag} />
