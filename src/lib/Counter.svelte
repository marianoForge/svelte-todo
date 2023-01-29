<script>
	export let initialCount = 0;
	export let maxCount = 10;
	let count = initialCount;

	function increment1() {
		if (count === maxCount) return;
		count += 1;
	}
</script>

<button on:click={increment1}>Clicks {count}</button>

<style>
	button {
		padding: 15px 20px;
	}
</style>
