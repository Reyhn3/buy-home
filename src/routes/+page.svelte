<script lang="ts">
	import {
		Button,
		Container,
		Col,
		Row,
		Input,
		Icon,
		Popover,
		Tooltip
	} from '@sveltestrap/sveltestrap';
</script>

<h1>Welcome to SvelteKit</h1>
<p>Visit <a href="https://kit.svelte.dev">kit.svelte.dev</a> to read the documentation</p>

<Container>
	<Row>
		<Col>
			<Input
				id="inp-price"
				type="number"
				placeholder="Pris"
				value="4500000"
				min="0"
				max="10000000"
				step="5000"
			/>
			<!-- <Tooltip target="inp-price" placement="right" delay="500">
				Detta är <strong>priset</strong> på bostaden.
			</Tooltip> -->
		</Col>
		<Col>
			<Icon id="icn-info-price" name="info-circle" />
			<Popover target="icn-info-price" placement="right" title="Pris" hideOnOutsideClick>
				Detta är <strong>priset</strong> på bostaden.
			</Popover>
		</Col>
	</Row>
	<Row>
		<Col>
			<Input
				id="inp-interest-rate"
				type="number"
				placeholder="Ränta"
				value="5.2"
				min="0"
				max="100"
				step="0.1"
			/>
		</Col>
		<Col>
			<Icon id="icn-info-interest-rate" name="info-circle" />
			<Popover target="icn-info-interest-rate" placement="right" title="Ränta" hideOnOutsideClick>
				Detta är <strong>räntesatsen</strong> för lånet.
			</Popover>
		</Col>
	</Row>
	<Row>
		<Col>
			<Input
				id="inp-deposit"
				type="number"
				placeholder="Insats"
				value="1000000"
				min="0"
				max="10000000"
				step="50000"
			/>
		</Col>
		<Col>
			<Icon id="icn-info-deposit" name="info-circle" />
			<Popover target="icn-info-deposit" placement="right" title="Insats" hideOnOutsideClick>
				Detta är <strong>kontantinsatsen</strong> till köpet.
			</Popover>
		</Col>
	</Row>
</Container>
