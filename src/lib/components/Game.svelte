<script context="module" lang="ts">
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

	export let width: number;
	export let height: number;
	export let mines: number;

	function forEachNeighbor(row: number, col: number, func: (i: number, j: number) => void) {
		for (let i = Math.max(row - 1, 0); i <= Math.min(row + 1, width - 1); i++) {
			for (let j = Math.max(col - 1, 0); j <= Math.min(col + 1, height - 1); j++) {
				if (i !== row || j !== col) {
					func(i, j);
				}
			}
		}
	}

	let counts: number[][];
	let alive: boolean;
	let states: State[][];
	startGame();

	function startGame() {
		alive = true;
		counts = Array.from({ length: height }, (_) => Array(width).fill(0));
		states = Array.from({ length: height }, (_) => Array(width).fill(State.Closed));
		placeMines(counts);
	}

	function incrementNeighbors(row: number, col: number) {
		forEachNeighbor(row, col, (i, j) => (counts[i][j] += 1));
	}

	function placeMines(counts: number[][]) {
		for (let n = 0; n < mines; n++) {
			while (true) {
				const i = random(0, counts.length - 1);
				const j = random(0, counts[0].length - 1);
				if (counts[i][j] !== Mine) {
					counts[i][j] = Mine;
					incrementNeighbors(i, j);
					break;
				}
			}
		}
	}

	function revealZeros(row: number, col: number) {
		states[row][col] = State.Revealed;

		if (counts[row][col] != 0) {
			return;
		}

		// reveal unrevealed neighbors neighbors
		forEachNeighbor(row, col, (i, j) => {
			if (states[i][j] !== State.Revealed) {
				revealZeros(i, j);
			}
		});
	}

	function hitMine(row: number, col: number) {
		return counts[row][col] === Mine;
	}

	function revealMap() {
		states = states.map((state) => state.fill(State.Revealed));
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

<Grid {counts} visible={states} {alive} on:reveal={reveal} on:flag={flag} />
