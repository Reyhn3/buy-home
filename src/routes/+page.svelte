<script lang="ts">
	import {
		Alert,
		Button,
		Container,
		Col,
		Row,
		Input,
		Icon,
		Popover,
		Tooltip
	} from '@sveltestrap/sveltestrap';
	import Currency from "./Currency.svelte";

	let locale = 'sv-SE'; //  https://github.com/libyal/libfwnt/wiki/Language-Code-identifiers

	const DEBT_RATIO_LIMIT = 4.5;
	const MIN_DEPOSIT_PERC = 0.15;


	// Household input
	$: income = (40 + 40) * 1000;


	// Property input
	$: price = 4500000;
	$: interestRate = 0.052;
	$: deposit = 1000000;


	// Calculated values
	$: yearlyIncome = income * 12;
	$: isMortgageLimitExceeded = loan > (yearlyIncome * DEBT_RATIO_LIMIT);
	$: loan = price - deposit;
	$: minDeposit = MIN_DEPOSIT_PERC * price;
	$: maxDeposit = Math.max(price, loan);
//TODO: Move arithmetics to functions
	$: interest = Math.round((loan * interestRate) / 12);
//TODO: Use the correct formula
	$: mortgage = Math.round((loan * 0.02) / 12);
</script>

<h1>Kalkylator för bostadsköp</h1>

<Container>

	<Row>
		<Col>
			<h2>Hushållet</h2>
		</Col>

	</Row>

	<Row>
		<Col xs="2">
			<Input
				id="inp-income"
				type="number"
				placeholder="Årsinkomst"
				bind:value={income}
				min="0"
				max="150000"
				step="5000"
			/>
		</Col>
		<Col>
			<Icon id="icn-info-income" name="info-circle" />
			<Popover target="icn-info-income" placement="right" title="Årsinkomst" hideOnOutsideClick>
				Detta är hushållets totala och gemensamma årsinkomst.
			</Popover>
		</Col>
		<Col xs="1">
			{yearlyIncome}
		</Col>
		<Col>
			<Icon id="icn-info-yearly-income" name="info-circle" />
			<Popover target="icn-info-yearly-income" placement="right" title="Årsinkomst" hideOnOutsideClick>
				Hushållets årliga inkomst.
			</Popover>
		</Col>
	</Row>


	<Row>
		<Col>
			<h2>Bostaden</h2>
		</Col>
	</Row>

	<Row>
		<Col xs="2">
			<Input
				id="inp-price"
				type="number"
				placeholder="Pris"
				bind:value={price}
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
		<Col xs="1">
			{loan}
		</Col>
		<Col>
			<Icon id="icn-info-loan" name="info-circle" />
			<Popover target="icn-info-loan" placement="right" title="Lån" hideOnOutsideClick>
				Detta är hur stort <strong>lånet</strong> blir.
			</Popover>
		</Col>
	</Row>

	<Row>
		<Col xs="2">
			<Input
				id="inp-deposit"
				type="number"
				placeholder="Insats"
				bind:value={deposit}
				min={minDeposit}
				max={maxDeposit}
				step="50000"
			/>
		</Col>
		<Col>
			<Icon id="icn-info-deposit" name="info-circle" />
			<Popover target="icn-info-deposit" placement="right" title="Insats" hideOnOutsideClick>
				Detta är <strong>kontantinsatsen</strong> till köpet.
			</Popover>
		</Col>
		<Col xs="1">
			{minDeposit}
		</Col>
		<Col>
			<Icon id="icn-info-min-deposit" name="info-circle" />
			<Popover target="icn-info-min-deposit" placement="right" title="Minsta insats" hideOnOutsideClick>
				<p>Detta är den <strong>minsta</strong> tillåtna insatsen.</p>
				<p>Kontantinsatsen måste vara minst 15% av köpeskillingen.</p>
			</Popover>
		</Col>
	</Row>

	<Row>
		<Col xs="2">
			<Input
				id="inp-interest-rate"
				type="number"
				placeholder="Ränta"
				bind:value={interestRate}
				min="0"
				max="100"
				step="0.1"
			/>
		</Col>
		<Col>
			<Icon id="icn-info-interest-rate" name="info-circle" />
			<Popover
				target="icn-info-interest-rate"
				placement="right"
				title="Ränta"
				hideOnOutsideClick
			>
				Detta är <strong>räntesatsen</strong> för lånet.
			</Popover>
		</Col>
	</Row>

	<Row>
		<Col>
			<h2>Kostnader</h2>
		</Col>
	</Row>

	<Row>
		<Col xs="2">
			<strong>Ränta:</strong>
		</Col>
		<Col>
			<Currency value={interest} {locale} />
		</Col>
	</Row>

	<Row>
		<Col xs="2">
			<strong>Amortering:</strong>
		</Col>
		<Col>
			<Currency value={mortgage} {locale} />
		</Col>
		<Col>
			{#if isMortgageLimitExceeded}
			<Alert color="warning" heading="Hög skuldkvot">
				<p>
					Om du tar ett nytt bolån, och lånet tillsammans med eventuella andra bolån som du redan har, är större än motsvarande 4,5 gånger din bruttoinkomst måste du amortera 1&nbsp;% extra per år på det nya lånet.
				</p>
				<p>
					<a href="https://www.konsumenternas.se/lan--betalningar/lan/bolan/amorteringskrav/" class="alert-link" target="_blank">
						Läs mer <Icon name="box-arrow-up-right" />
					</a>
				</p>
			</Alert>
			{/if}
		</Col>
	</Row>
</Container>
