<script>
	import { onMount } from 'svelte'
	
	export let result_json

	let textarea
	
	onMount(() => {
		textarea.select()
		document.execCommand('copy')
	})
</script>

<textarea bind:value={result_json} bind:this={textarea}></textarea>
