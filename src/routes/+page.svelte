<script lang="ts">
	import {
		Alert,
		Container,
		Col,
		Row,
		Input,
		Icon,
		Popover,
		Table
	} from '@sveltestrap/sveltestrap';
	import FormattedNumber from './FormattedNumber.svelte';
	import Details from './Details.svelte';


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
	 * @param {number} d The size of the deposit
	 */
	const calcMaxLoan = (d) => {
		return Math.round(d / 0.15) - d;
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
	const calcMortgageRate = (p, l, i) => {
		var quotient = l / p;
		var mortgageRate = 0;

		if (quotient > 0.7) mortgageRate = 0.02;
		else if (quotient > 0.5) mortgageRate = 0.01;

		var debtRatio = calcDebtRatio(l, i);
		if (debtRatio > DEBT_RATIO_LIMIT) mortgageRate += 0.01;

		return mortgageRate;
	};

	/**
	 * @param {number} p The price of the house
	 * @param {number} l The size of the loan
	 * @param {number} i The monthly household income
	 */
	const calcMortgage = (p, l, i) => {
		var mortgageRate = calcMortgageRate(p, l, i);
		return Math.round((l * mortgageRate) / 12);
	};

	/**
	 * @param {number} i The monthly interest amount
	 */
	const calcDeduction = (i) => {
		var yearlyInterest = i * 12;

		if (yearlyInterest < DEDUCTION_LIMIT) {
			return Math.round((DEDUCTION_FIRST * yearlyInterest) / 12);
		}

		return Math.round(
			(DEDUCTION_FIRST * DEDUCTION_LIMIT +
				DEDUCTION_REST * (yearlyInterest - DEDUCTION_LIMIT)) /
				12
		);
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
	$: maxLoan = calcMaxLoan(deposit);
	$: debtRatio = calcDebtRatio(loan, income);
	$: isMortgageLimitExceeded = debtRatio > DEBT_RATIO_LIMIT;
	$: minDeposit = MIN_DEPOSIT_PERC * price;
	$: maxDeposit = Math.max(price, loan);
	$: depositPerc = Math.round((deposit / price + Number.EPSILON) * 100) / 100;
	$: interest = calcInterest(loan, interestRate);
	$: mortgageRate = calcMortgageRate(price, loan, income);
	$: mortgage = calcMortgage(price, loan, income);
	$: deduction = calcDeduction(interest);
	$: sumWithoutDeduction = calcSum(interest, mortgage, upkeep, tax, 0);
	$: sumWithDeduction = calcSum(interest, mortgage, upkeep, tax, deduction);
</script>

<Container>
	<Row>
		<Col>
			<h2>Hushållet</h2>
		</Col>
	</Row>

	<Row>
		<Col>
			<span class="label">
				Inkomst
				<Icon id="icn-info-income" name="info-circle" style="info" />
			</span>
			<Popover target="icn-info-income" placement="right" title="Inkomst" hideOnOutsideClick>
				Detta är hushållets gemensamma inkomst per månad.
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
	</Row>
	<Row>
		<Col>
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
			<Details id="yearly-income" label="Årsinkomst" value={yearlyIncome}>
				<span slot="description"> Hushållets årliga inkomst. </span>
			</Details>
		</Col>
	</Row>

	<Row>
		<Col>
			<h2>Bostaden</h2>
		</Col>
	</Row>

	<Row>
		<Col>
			<span class="label">
				Pris
				<Icon id="icn-info-price" name="info-circle" style="info" />
			</span>
			<Popover target="icn-info-price" placement="right" title="Pris" hideOnOutsideClick>
				Detta är <strong>priset</strong> på bostaden.
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
	</Row>
	<Row>
		<Col>
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
			<Details id="min-deposit" label="Minsta insats" value={minDeposit}>
				<span slot="description">
					<p>Detta är den <strong>minsta</strong> tillåtna insatsen.</p>
					<p>Kontantinsatsen måste vara minst 15% av köpeskillingen.</p>
				</span>
			</Details>
		</Col>
	</Row>

	<Row>
		<Col>
			<span class="label">
				Insats
				<Icon id="icn-info-deposit" name="info-circle" style="info" />
			</span>
			<Popover target="icn-info-deposit" placement="right" title="Insats" hideOnOutsideClick>
				Detta är <strong>kontantinsatsen</strong>.
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
	</Row>
	<Row>
		<Col>
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
			<Details id="loan" label="Lån" value={loan}>
				<span slot="description"> Lånets storlek. </span>
			</Details>
		</Col>
		<Col>
			<Details id="deposit-perc" label="Procent" value={depositPerc} style="percent">
				<span slot="description"> Insatsen i procent av bostadens pris. </span>
			</Details>
		</Col>
		<Col>
			<Details
				id="debt-ratio"
				label="Skuldkvot"
				value={debtRatio}
				style="decimal"
				decimals="1"
			>
				<span slot="description">
					Lånets storlek i förhållande till hushållets bruttoårsinkomst.
				</span>
			</Details>
		</Col>
		<Col>
			<Details id="max-loan" label="Lånetak" value={maxLoan}>
				<span slot="description"> Maximalt tillåtet lån. </span>
			</Details>
		</Col>
		<Col>
			<Details id="mortgage-rate" label="Amortering" value={mortgageRate} style="percent">
				<span slot="description"> Procentsatsen för amortering. </span>
			</Details>
		</Col>
	</Row>

	<Row>
		<Col>
			<span class="label">
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
	<Row>
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
	<Row>
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
			<span class="label">
				Driftkostnad
				<Icon id="icn-info-upkeep" name="info-circle" style="info" />
			</span>
			<Popover
				target="icn-info-upkeep"
				placement="right"
				title="Driftkostnad"
				hideOnOutsideClick
			>
				Bostadens driftkostnad, exempelvis:
				<ul>
					<li>El</li>
					<li>Vatten</li>
					<li>Uppvärmning</li>
					<li>Försäkring</li>
					<li>Avlopp</li>
					<li>Sophantering</li>
					<li>Snöröjning</li>
					<li>Månadsavgift</li>
				</ul>
			</Popover>
		</Col>
	</Row>
	<Row>
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
	<Row>
		<Col>
			<Input id="inp-upkeep" type="range" bind:value={upkeep} min="0" max="25000" step="50" />
		</Col>
	</Row>

	<Row>
		<Col>
			<span class="label">
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
	<Row>
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
	<Row>
		<Col>
			<Input id="inp-tax" type="range" bind:value={tax} min="0" max="10000" step="50" />
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
						<td>Ränta</td>
						<td><FormattedNumber value={interest} /></td>
					</tr>
					<tr>
						<td>Amortering</td>
						<td>
							<FormattedNumber value={mortgage} />
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
						<td>Avdrag</td>
						<td><FormattedNumber value={deduction} /></td>
					</tr>
					<tr>
						<td style="font-weight:bold;">Summa</td>
						<td><FormattedNumber value={sumWithoutDeduction} --font-weight="bold" /></td
						>
					</tr>
					<tr>
						<td style="color: slategray;">Summa (efter avdrag)</td>
						<td><FormattedNumber value={sumWithDeduction} --color="slategray" /></td>
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
</style>
