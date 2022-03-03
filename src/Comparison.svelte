<script lang="ts">
	export let optionLeft: string = ""
	export let optionRight: string = ""

	import { createEventDispatcher } from "svelte"

    const dispatch = createEventDispatcher()
	const choseLeft = (leftIsTrue) => {
		dispatch("choseleft", leftIsTrue)
	}
</script>

<main>
	<div class="option" id="leftOption" on:click={ () => choseLeft(true) }>{optionLeft}</div>
	<div class="option" id="rightOption" on:click={ () => choseLeft(false) }>{optionRight}</div>
</main>

<style lang="scss">
	main {
		position: fixed;
		width: 100%;
		height: 100%;
		z-index: 1;
	}

	.option {
		position: absolute;
		width: 50%;
		height: 100%;
		font-size: 6rem;
		display: grid;
		place-items: center;
	}

	#leftOption {
		background-color: var(--textcolor);
		color: #111;
	}
	#rightOption {
		background-color: #111;
		color: var(--textcolor);
		left: 50%;
	}
</style>