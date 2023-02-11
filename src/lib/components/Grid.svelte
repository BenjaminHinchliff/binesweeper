<script lang="ts">
	export let width: number;
	export let height: number;
</script>

<div class="grid" style:grid-template-columns="repeat({width}, 1fr [col-start])">
	{#each Array(width) as _, i}
		{#each Array(height) as _, j}
			<button class="grid-cell" on:click={console.log} on:contextmenu|preventDefault={console.log}
				>{i}, {j}</button
			>
		{/each}
	{/each}
</div>

<style lang="scss">
	.grid {
		display: grid;
	}

	.grid-cell {
		font-size: 4vw;
		aspect-ratio: 1;
	}
</style>
