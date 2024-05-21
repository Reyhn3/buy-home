<script>
	import { Col, Container, Icon, Popover, Row } from '@sveltestrap/sveltestrap';
	import FormattedNumber from './FormattedNumber.svelte';

	export let id;
	export let label;
	export let value;
	export let style;
	export let decimals;
</script>

<Container>
	<Row>
		<Col>
			<span class="label">
				{label}
				<Icon id="icn-info-{id}" name="info-circle" style="info" />
			</span>
			<!-- <Tooltip target="lbl" placement="right" delay="500">
				{description}
			</Tooltip> -->
			<Popover target="icn-info-{id}" placement="right" title={label} hideOnOutsideClick>
				<slot name="description" />
			</Popover>
		</Col>
	</Row>
	<Row>
		<Col>
			<FormattedNumber {value} {style} {decimals} --font-size="0.7em" --color="slategray" />
		</Col>
	</Row>
</Container>

<style>
	.label {
		color: slategray;
		font-family: 'Comic Sans MS';
		font-size: 0.7em;
		font-weight: bold;
	}
</style>
