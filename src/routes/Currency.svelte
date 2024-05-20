<script>
	export let value;
	export let locale = 'sv-SE';

	$: formatted = value.toLocaleString(locale, {
		style: 'currency',
		currency: 'SEK',
		minimumFractionDigits: 0
	});
</script>

<span>{formatted}</span>

<style>
	span {
		font-variant-numeric: tabular-nums;
		font-size: 1.2em;
		font-weight: normal;
	}
</style>
