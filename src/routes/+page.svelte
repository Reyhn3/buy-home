<script lang="ts">
	import {
		Alert,
		Container,
		Col,
		Row,
		Input,
		Icon,
		Popover,
		Table,
		Tooltip
	} from '@sveltestrap/sveltestrap';
	import Currency from './Currency.svelte';


	const DEBT_RATIO_LIMIT = 4.5;
	const MIN_DEPOSIT_PERC = 0.15;
	const DEDUCTION_LIMIT = 200000;
	const DEDUCTION_FIRST = 0.3;
	const DEDUCTION_REST = 0.21;


	//TODO: Extract functions from here
	/**
	 * @param {number} p The price of the house
	 * @param {number} d The size of the deposit
	 */
	const calcLoan = (p, d) => {
		return Math.max(0, p - d);
	};

	/**
	 * @param {number} l The size of the loan
	 * @param {number} ir The interest rate
	 */
	const calcInterest = (l, ir) => {
		return Math.round((l * ir) / 100 / 12);
	};

	/**
	 * @param {number} l The size of the loan
	 * @param {number} i The monthly household income
	 */
	const calcDebtRatio = (l, i) => {
		var yearlyIncome = i * 12;
		var quotient = l / yearlyIncome;
		return quotient;
	};

	/**
	 * @param {number} p The price of the house
	 * @param {number} l The size of the loan
	 * @param {number} i The monthly household income
	 */
	const calcMortgage = (p, l, i) => {
		var quotient = l / p;
		var mortgageRate = 0;

		if (quotient > 0.7) mortgageRate = 0.02;
		else if (quotient > 0.5) mortgageRate = 0.01;

		var debtRatio = calcDebtRatio(l, i);
		if (debtRatio > DEBT_RATIO_LIMIT) mortgageRate += 0.01;

		return Math.round((l * mortgageRate) / 12);
	};

	/**
	 * @param {number} i The monthly interest amount
	 */
	const calcDeduction = (i) => {
		var yearlyInterest = i * 12;

		if (yearlyInterest < DEDUCTION_LIMIT) {
			return Math.round(DEDUCTION_FIRST * yearlyInterest / 12);
		}

		return Math.round((DEDUCTION_FIRST * DEDUCTION_LIMIT + DEDUCTION_REST * (yearlyInterest - DEDUCTION_LIMIT)) / 12);
	};

	/**
	 * @param {number} i The monthly interest amount
	 * @param {number} m The monthly mortgage amount
	 * @param {number} u The monthly upkeep cost
	 * @param {number} t The yearly property tax
	 * @param {number} d The monthly deduction amount
	 */
	const calcSum = (i, m, u, t, d) => {
		return Math.round(i + m + u + t / 12 - d);
	};


	// Household input
	$: income = (40 + 40) * 1000;

	// Property input
	$: price = 4500000;
	$: interestRate = 5.2;
	$: deposit = 1000000;
	$: upkeep = 4500;
	$: tax = 9525;

	// Calculated values
	$: yearlyIncome = income * 12;
	$: loan = calcLoan(price, deposit);
	$: isMortgageLimitExceeded = calcDebtRatio(loan, income) > DEBT_RATIO_LIMIT;
	$: minDeposit = MIN_DEPOSIT_PERC * price;
	$: maxDeposit = Math.max(price, loan);
	$: interest = calcInterest(loan, interestRate);
	$: mortgage = calcMortgage(price, loan, income);
	$: deduction = calcDeduction(interest);
	$: sumWithoutDeduction = calcSum(interest, mortgage, upkeep, tax, 0);
	$: sumWithDeduction = calcSum(interest, mortgage, upkeep, tax, deduction);
</script>

<h1>Kalkylator för bostadsköp</h1>

<Container>
	<Row>
		<Col>
			<h2>Hushållet</h2>
		</Col>
	</Row>

	<Row>
		<Col>
			<span id="lbl-income" class="label">
				Inkomst
				<Icon id="icn-info-income" name="info-circle" style="info" />
			</span>
			<!-- <Tooltip target="lbl-income" placement="right" delay="500">
				Detta är hushållets totala och gemensamma årsinkomst.
			</Tooltip> -->
			<Popover target="icn-info-income" placement="right" title="Inkomst" hideOnOutsideClick>
				Detta är hushållets gemensamma inkomst per månad.
			</Popover>
		</Col>
		<Col>
			<span id="lbl-yearly-income" class="label">
				Årsinkomst
				<Icon id="icn-info-yearly-income" name="info-circle" style="info" />
			</span>
			<Popover
				target="icn-info-yearly-income"
				placement="right"
				title="Årsinkomst"
				hideOnOutsideClick
			>
				Hushållets årliga inkomst.
			</Popover>
		</Col>
	</Row>
	<Row>
		<Col>
			<Input
				id="inp-income"
				type="number"
				placeholder="Månadsinkomst"
				bind:value={income}
				min="0"
				max="150000"
				step="5000"
			/>
		</Col>
		<Col>
			<Currency value={yearlyIncome} />
		</Col>
	</Row>
	<Row>
		<Col xs="6">
			<Input
				id="inp-income"
				type="range"
				bind:value={income}
				min="0"
				max="150000"
				step="5000"
			/>
		</Col>
	</Row>

	<Row>
		<Col>
			<h2>Bostaden</h2>
		</Col>
	</Row>

	<Row>
		<Col>
			<span id="lbl-price" class="label">
				Pris
				<Icon id="icn-info-price" name="info-circle" style="info" />
			</span>
			<Popover target="icn-info-price" placement="right" title="Pris" hideOnOutsideClick>
				Detta är <strong>priset</strong> på bostaden.
			</Popover>
		</Col>
		<Col>
			<span id="lbl-price" class="label">
				Lån
				<Icon id="icn-info-loan" name="info-circle" />
			</span>
			<Popover target="icn-info-loan" placement="right" title="Lån" hideOnOutsideClick>
				Detta är hur stort <strong>lånet</strong> blir.
			</Popover>
		</Col>
	</Row>
	<Row>
		<Col>
			<Input
				id="inp-price"
				type="number"
				placeholder="Pris"
				bind:value={price}
				min="0"
				max="10000000"
				step="5000"
			/>
		</Col>
		<Col>
			<Currency value={loan} />
		</Col>
		<!--TODO: Add skuldkvot-->
	</Row>
	<Row>
		<Col xs="6">
			<Input
				id="inp-price"
				type="range"
				bind:value={price}
				min="0"
				max="10000000"
				step="5000"
			/>
		</Col>
	</Row>

	<Row>
		<Col>
			<span id="lbl-deposit" class="label">
				Insats
				<Icon id="icn-info-deposit" name="info-circle" style="info" />
			</span>
			<Popover target="icn-info-deposit" placement="right" title="Insats" hideOnOutsideClick>
				Detta är <strong>kontantinsatsen</strong>.
			</Popover>
		</Col>
		<Col>
			<span id="lbl-deposit" class="label">
				Minsta insats
				<Icon id="icn-info-min-deposit" name="info-circle" />
			</span>
			<Popover
				target="icn-info-min-deposit"
				placement="right"
				title="Minsta insats"
				hideOnOutsideClick
			>
				<p>Detta är den <strong>minsta</strong> tillåtna insatsen.</p>
				<p>Kontantinsatsen måste vara minst 15% av köpeskillingen.</p>
			</Popover>
		</Col>
	</Row>
	<Row>
		<Col>
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
			<Currency value={minDeposit} />
		</Col>
		<!--TODO: Add procentsats för insatsen-->
	</Row>
	<Row>
		<Col xs="6">
			<Input
				id="inp-deposit"
				type="range"
				bind:value={deposit}
				min={minDeposit}
				max={maxDeposit}
				step="25000"
			/>
		</Col>
	</Row>

	<Row>
		<Col>
			<span id="lbl-interest-rate" class="label">
				Ränta
				<Icon id="icn-info-interest-rate" name="info-circle" style="info" />
			</span>
			<Popover
				target="icn-info-interest-rate"
				placement="right"
				title="Ränta"
				hideOnOutsideClick
			>
				Detta är <strong>räntesatsen</strong> på bolånet.
			</Popover>
		</Col>
	</Row>
	<Row cols="2">
		<Col>
			<Input
				id="inp-interest-rate"
				type="number"
				placeholder="Ränta"
				bind:value={interestRate}
				min="0"
				max="25"
				step="0.1"
			/>
		</Col>
	</Row>
	<Row cols="2">
		<Col>
			<Input
				id="inp-interest-rate"
				type="range"
				bind:value={interestRate}
				min="0"
				max="25"
				step="0.1"
			/>
		</Col>
	</Row>

	<Row>
		<Col>
			<span id="lbl-upkeep" class="label">
				Driftkostnad
				<Icon id="icn-info-upkeep" name="info-circle" style="info" />
			</span>
			<Popover
				target="icn-info-upkeep"
				placement="right"
				title="Driftkostnad"
				hideOnOutsideClick
			>
				Bostadens driftkostnad.
			</Popover>
		</Col>
	</Row>
	<Row cols="2">
		<Col>
			<Input
				id="inp-upkeep"
				type="number"
				placeholder="Driftkostnad"
				bind:value={upkeep}
				min="0"
				max="25000"
				step="50"
			/>
		</Col>
	</Row>
	<Row cols="2">
		<Col>
			<Input
				id="inp-upkeep"
				type="range"
				bind:value={upkeep}
				min="0"
				max="25000"
				step="50"
			/>
		</Col>
	</Row>

	<Row>
		<Col>
			<span id="lbl-tax" class="label">
				Fastighetsskatt
				<Icon id="icn-info-tax" name="info-circle" style="info" />
			</span>
			<Popover
				target="icn-info-tax"
				placement="right"
				title="Fastighetsskatt"
				hideOnOutsideClick
			>
				Fastighetsskatt
			</Popover>
		</Col>
	</Row>
	<Row cols="2">
		<Col>
			<Input
				id="inp-tax"
				type="number"
				placeholder="Fastighetsskatt"
				bind:value={tax}
				min="0"
				max="10000"
				step="50"
			/>
		</Col>
	</Row>
	<Row cols="2">
		<Col>
			<Input
				id="inp-tax"
				type="range"
				bind:value={tax}
				min="0"
				max="10000"
				step="50"
			/>
		</Col>
	</Row>

	<Row>
		<Col>
			<h2>Kostnader</h2>
		</Col>
	</Row>

	<Row cols="2">
		<Col>
			<Table borderless hover>
				<tbody>
					<tr>
						<th>Ränta</th>
						<td><Currency value={interest} /></td>
					</tr>
					<tr>
						<th>Amortering</th>
						<td>
							<Currency value={mortgage} />
							{#if isMortgageLimitExceeded}
								<Alert color="warning" heading="Hög skuldkvot">
									<p>
										Om du tar ett nytt bolån, och lånet tillsammans med
										eventuella andra bolån som du redan har, är större än
										motsvarande 4,5 gånger din bruttoinkomst måste du amortera
										1&nbsp;% extra per år på det nya lånet.
									</p>
									<p>
										<a
											href="https://www.konsumenternas.se/lan--betalningar/lan/bolan/amorteringskrav/"
											class="alert-link"
											target="_blank"
										>
											Läs mer&nbsp;<Icon name="box-arrow-up-right" />
										</a>
									</p>
								</Alert>
							{/if}
						</td>
					</tr>
					<tr>
						<th>Avdrag</th>
						<td><Currency value={deduction} /></td>
					</tr>
					<tr>
						<th>Summa</th>
						<td><Currency value={sumWithoutDeduction} /></td>
					</tr>
					<tr>
						<th>Summa (efter avdrag)</th>
						<td><Currency value={sumWithDeduction} /></td>
					</tr>
				</tbody>
			</Table>
		</Col>
	</Row>
</Container>

<style>
	h2 {
		font-size: 1.2em;
	}

	.label {
		color: slategray;
		font-family: 'Comic Sans MS';
		font-size: 0.7em;
		font-weight: bold;
	}
</style>
