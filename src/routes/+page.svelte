<script lang="ts">
	import Game from '$lib/components/Game.svelte';
	import mineImg from '$lib/assets/mine-plain.svg';
</script>

<svelte:head>
	<title>Binesweeper</title>
</svelte:head>

<h1 id="title">Binesweeper <img id="logo" alt="naval mine" src={mineImg} /></h1>
<Game />

<style lang="scss">
	#title {
		text-align: center;
		margin-bottom: 1rem;
	}

	@media only screen and (prefers-color-scheme: dark) {
		#logo {
			filter: invert(100%);
		}
	}
</style>
