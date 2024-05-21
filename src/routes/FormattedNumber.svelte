<script>
	export let value;
	export let style = "currency";
	export let decimals = 0;
	export let locale = "sv-SE"; // https://github.com/libyal/libfwnt/wiki/Language-Code-identifiers

	let format = {
		style: style,
		currency: "SEK",
		minimumFractionDigits: decimals,
		maximumFractionDigits: Math.max(0, decimals)
	};

	$: formatted = value.toLocaleString(locale, format);
</script>

<span>{formatted}</span>

<style>
	span {
		color: var(--color);
		font-variant-numeric: tabular-nums;
		font-size: var(--font-size);
		font-weight: var(--font-weight);
	}
</style>
