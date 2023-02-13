<script context="module" lang="ts">
	export interface Difficulty {
		name: string;
		width: number;
		height: number;
		mines: number;
	}

	export enum State {
		Revealed,
		Flagged,
		Closed
	}

	export const Mine = Infinity;
</script>

<script lang="ts">
	import { random } from 'underscore';

	import Grid from './Grid.svelte';
	import Options from './Options.svelte';

	const difficulties: Difficulty[] = [
		{ name: 'Beginner', width: 9, height: 9, mines: 10 },
		{ name: 'Intermediate', width: 16, height: 16, mines: 40 },
		{ name: 'Expert', width: 30, height: 16, mines: 99 }
	];

	let difficulty = Math.floor(difficulties.length / 2);
	let alive: boolean;
	let counts: number[][];
	let states: State[][];

	$: startGame(difficulties[difficulty]);

	function startGame({ width, height, mines }: Difficulty) {
		alive = true;
		counts = Array.from({ length: height }, (_) => Array(width).fill(0));
		states = Array.from({ length: height }, (_) => Array(width).fill(State.Closed));
		placeMines(counts, width, height, mines);
	}

	function forEachNeighbor(
		row: number,
		col: number,
		width: number,
		height: number,
		func: (i: number, j: number) => void
	) {
		for (let i = Math.max(row - 1, 0); i <= Math.min(row + 1, height - 1); i++) {
			for (let j = Math.max(col - 1, 0); j <= Math.min(col + 1, width - 1); j++) {
				if (i !== row || j !== col) {
					func(i, j);
				}
			}
		}
	}

	function incrementNeighbors(row: number, col: number, width: number, height: number) {
		forEachNeighbor(row, col, width, height, (i, j) => (counts[i][j] += 1));
	}

	function placeMines(counts: number[][], width: number, height: number, mines: number) {
		for (let n = 0; n < mines; n++) {
			while (true) {
				const i = random(0, height - 1);
				const j = random(0, width - 1);
				if (counts[i][j] !== Mine) {
					counts[i][j] = Mine;
					incrementNeighbors(i, j, width, height);
					break;
				}
			}
		}
	}

	$: width = counts[0].length;
	$: height = counts.length;

	function revealZeros(row: number, col: number) {
		states[row][col] = State.Revealed;

		if (counts[row][col] != 0) {
			return;
		}

		// reveal closed neighbors
		forEachNeighbor(row, col, width, height, (i, j) => {
			if (states[i][j] === State.Closed) {
				revealZeros(i, j);
			}
		});
	}

	function hitMine(row: number, col: number) {
		return counts[row][col] === Mine;
	}

	function revealMap() {
		for (let i = 0; i < height; i++) {
			for (let j = 0; j < width; j++) {
				if (states[i][j] !== State.Flagged || counts[i][j] !== Mine) {
					states[i][j] = State.Revealed;
				}
			}
		}
	}

	// check if any closed cells still exist without mines
	function hasWon(): boolean {
		for (let i = 0; i < height; i++) {
			for (let j = 0; j < width; j++) {
				if (counts[i][j] !== Mine && states[i][j] !== State.Revealed) {
					return false;
				}
			}
		}
		return true;
	}

	// flag all bombs (for if won but didn't mark them)
	function flagBombs() {
		for (let i = 0; i < height; i++) {
			for (let j = 0; j < width; j++) {
				if (counts[i][j] === Mine) {
					states[i][j] = State.Flagged;
				}
			}
		}
	}

	function reveal(event: CustomEvent<{ row: number; col: number }>) {
		const { row, col } = event.detail;
		// don't reveal flagged squares
		if (states[row][col] === State.Flagged) {
			return;
		}

		revealZeros(row, col);
		states[row][col] = State.Revealed;
		if (hitMine(row, col)) {
			alive = false;
			revealMap();
		} else if (hasWon()) {
			alive = false;
			flagBombs();
		}
	}

	function flag(event: CustomEvent<{ row: number; col: number }>) {
		const { row, col } = event.detail;
		if (states[row][col] === State.Flagged) {
			states[row][col] = State.Closed;
		} else {
			states[row][col] = State.Flagged;
		}
	}
</script>

<div class="game">
	<Grid {counts} visible={states} {alive} on:reveal={reveal} on:flag={flag} />
	<Options on:start={() => startGame(difficulties[difficulty])} bind:difficulty {difficulties} />
</div>

<style>
	.game {
		--width: min(100vw, 90vh);
		max-width: var(--width);
		margin: auto;
	}
</style>
