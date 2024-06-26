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
	const PANTBREV_PERC = 0.02;
	const PANTBREV_FEE = 375;
	const LAGFART_PERC = 0.015;
	const LAGFART_FEE = 825;

	//TODO: Extract functions from here
	/**
	 * Calculates the size of the loan.
	 *
	 * @param {number} p The price of the house.
	 * @param {number} d The size of the deposit.
	 */
	const calcLoan = (p: number, d: number) => {
		return Math.max(0, p - d);
	};

	/**
	 * Calculates the max size of loan allowed.
	 *
	 * @param {number} d The size of the deposit.
	 */
	const calcMaxLoan = (d: number) => {
		return Math.round(d / 0.15) - d;
	};

	/**
	 * Calculates the monthly interest cost.
	 *
	 * @param {number} l The size of the loan.
	 * @param {number} ir The interest rate.
	 * @param {number} d The **monthly** deduction amount.
	 */
	const calcInterest = (l: number, ir: number, d: number) => {
		return Math.round((l * ir) / 100 / 12 + d);
	};

	/**
	 * Calculates the debt ratio.
	 *
	 * @param {number} l The size of the loan.
	 * @param {number} i The monthly household income.
	 */
	const calcDebtRatio = (l: number, i: number) => {
		var yearlyIncome = i * 12;
		var quotient = l / yearlyIncome;
		return quotient;
	};

	/**
	 * Calculates the mortgage rate.
	 *
	 * @param {number} p The price of the house.
	 * @param {number} l The size of the loan.
	 * @param {number} i The monthly household income.
	 */
	const calcMortgageRate = (p: number, l: number, i: number) => {
		var quotient = l / p;
		var mortgageRate = 0;

		if (quotient > 0.7) mortgageRate = 0.02;
		else if (quotient > 0.5) mortgageRate = 0.01;

		var debtRatio = calcDebtRatio(l, i);
		if (debtRatio > DEBT_RATIO_LIMIT) mortgageRate += 0.01;

		return mortgageRate;
	};

	/**
	 * Calculates the monthly mortgage.
	 *
	 * @param {number} p The price of the house.
	 * @param {number} l The size of the loan.
	 * @param {number} i The monthly household income.
	 */
	const calcMortgage = (p: number, l: number, i: number) => {
		var mortgageRate = calcMortgageRate(p, l, i);
		return Math.round((l * mortgageRate) / 12);
	};

	/**
	 * Calculates the monthly tax deduction.
	 *
	 * @param {number} i The monthly interest amount.
	 */
	const calcDeduction = (i: number) => {
		var yearlyInterest = i * 12;

		if (yearlyInterest < DEDUCTION_LIMIT) {
			return -1 * Math.round((DEDUCTION_FIRST * yearlyInterest) / 12);
		}

		return (
			-1 *
			Math.round(
				(DEDUCTION_FIRST * DEDUCTION_LIMIT +
					DEDUCTION_REST * (yearlyInterest - DEDUCTION_LIMIT)) /
					12
			)
		);
	};

	/**
	 * Calculates the total monthly cost of owning the house.
	 *
	 * @param {number} i The **monthly** interest amount.
	 * @param {number} m The **monthly** mortgage amount.
	 * @param {number} u The **monthly** upkeep cost.
	 * @param {number} t The **yearly** property tax.
	 * @param {number} d The **monthly** deduction amount.
	 */
	const calcSum = (i: number, m: number, u: number, t: number, d: number) => {
		return Math.round(i + m + u + t / 12 - Math.abs(d));
	};

	/**
	 * Calculates the size of new pantbrev needed.
	 *
	 * @param {number} l The size of the loan.
	 * @param {number} b The size of the existing pantbrev.
	 */
	const calcPantbrev = (l: number, b: number) => {
		var additional = l - b;
		console.warn("pantbrev " + additional)
		if (additional > 0) {
			return Math.round(PANTBREV_PERC * additional + PANTBREV_FEE);
		}

		return 0;
	};

	/**
	 * Calculates the size of the lagfart.
	 *
	 * @param {number} p The price of the house.
	 */
	const calcLagfart = (p: number) => {
		return Math.round(LAGFART_PERC * p + LAGFART_FEE);
	};

	/**
	 * Calculates the sum of all selling costs. These are the costs associated with
	 * selling the current residence, like realtor fees etc.
	 *
	 * @param {number[]} arr The selling costs to sum up.
	 */
	const calcSellingCosts = (arr: number[]) => {
		var total = 0;
		for (var i = 0; i < arr.length; i++) {
    		total += arr[i];
  		}
  		return Math.round(total);
	};

	/**
	 * Calculates the sum of all moving costs. These are the costs associated with
	 * the actual moving, like cleaning, renting trucks etc.
	 *
	 * @param {number[]} arr The moving costs to sum up.
	 */
	const calcMovingCosts = (arr: number[]) => {
		var total = 0;
		for (var i = 0; i < arr.length; i++) {
    		total += arr[i];
  		}
  		return Math.round(total);
	};

	/**
	 * Calculates the sum of all buying costs.
	 *
	 * @param {number[]} arr The buying costs to sum up.
	 */
	const calcBuyingCost = (arr: number[]) => {
		var total = 0;
		for (var i = 0; i < arr.length; i++) {
    		total += arr[i];
  		}
  		return Math.round(total);
	};

	// Household input
	$: income = (40 + 40) * 1000;

	// Property input
	$: price = 4500000;
	$: interestRate = 5.2;
	$: deposit = price * MIN_DEPOSIT_PERC;
	$: upkeep = 4500;
	$: tax = 9525;
	$: pawn = 2000000;

	// Variable values
	$: realtorFee = 0
	$: marketing = 0
	$: stylingFee = 0
	$: doubleLiving = 0
	$: transferFee = 0
	$: membershipFee = 0
	$: addressChange = 1000
	$: movingBoxes = 2100
	$: movingHelp = 13500
	$: movingCleaning = 3200

	// Calculated values
	$: yearlyIncome = income * 12;
	$: loan = calcLoan(price, deposit);
	$: maxLoan = calcMaxLoan(deposit);
	$: debtRatio = calcDebtRatio(loan, income);
	$: isMortgageLimitExceeded = debtRatio > DEBT_RATIO_LIMIT;
	$: minDeposit = MIN_DEPOSIT_PERC * price;
	$: maxDeposit = Math.max(price, loan);
	$: depositPerc = Math.round((deposit / price + Number.EPSILON) * 100) / 100;
	$: interest = calcInterest(loan, interestRate, 0);
	$: mortgageRate = calcMortgageRate(price, loan, income);
	$: mortgage = calcMortgage(price, loan, income);
	$: deduction = calcDeduction(interest);
	$: sumWithoutDeduction = calcSum(interest, mortgage, upkeep, tax, 0);
	$: sumWithDeduction = calcSum(interest, mortgage, upkeep, tax, deduction);
	$: interestWithDeduction = calcInterest(loan, interestRate, deduction);
	$: pantbrev = calcPantbrev(loan, pawn);
	$: lagfart = calcLagfart(price);
	$: sumSelling = calcSellingCosts([realtorFee, marketing, stylingFee, doubleLiving]);
	$: sumMoving = calcMovingCosts([transferFee, membershipFee, addressChange, movingBoxes, movingHelp, movingCleaning]);
	$: sumBuying = calcBuyingCost([pantbrev, lagfart, sumSelling, sumMoving]);
</script>

<div class="header">
	<Container>
		<Row>
			<Col>
				<span class="label">
					Månadskostnad&nbsp;
					<FormattedNumber value={sumWithoutDeduction} --font-weight="bold" />
					(<FormattedNumber value={sumWithDeduction} />&nbsp;efter avdrag)
				</span>
			</Col>
		</Row>
	</Container>
</div>

<ul class="nav nav-tabs" role="tablist">
	<li class="nav-item" role="presentation">
		<button
			class="nav-link active"
			id="living-tab"
			data-bs-toggle="tab"
			data-bs-target="#living-tab-pane"
			type="button"
			role="tab"
			aria-controls="living-tab-pane"
			aria-selected="true">
			Boendekostnad
		</button>
	</li>
	<li class="nav-item" role="presentation">
		<button
			class="nav-link"
			id="buying-tab"
			data-bs-toggle="tab"
			data-bs-target="#buying-tab-pane"
			type="button"
			role="tab"
			aria-controls="buying-tab-pane"
			aria-selected="false">
			Köpekostnad
		</button>
	</li>
</ul>

<div class="tab-content">
	<div
		class="tab-pane fade show active"
		id="living-tab-pane"
		role="tabpanel"
		aria-labelledby="living-tab"
		tabindex="0">
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

	<Row cols="3">
		<Col>
			<Table borderless hover>
				<tbody>
					<tr>
						<td>Ränta</td>
						<td class="text-end"><FormattedNumber value={interest} /></td>
					</tr>
					<tr>
						<td>Amortering</td>
						<td class="text-end">
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
						<td style="font-weight:bold;">Summa</td>
						<td class="text-end"><FormattedNumber value={sumWithoutDeduction} --font-weight="bold" /></td
						>
					</tr>
					<tr>
						<td style="color: slategray;">Avdrag</td>
						<td class="text-end"><FormattedNumber value={deduction} --color="slategray" /></td>
					</tr>
					<tr>
						<td style="color: slategray;">Ränta (efter avdrag)</td>
						<td class="text-end"><FormattedNumber value={interestWithDeduction} --color="slategray" /></td>
					</tr>
					<tr>
						<td style="color: slategray;font-weight:bold;">Summa (efter avdrag)</td>
						<td class="text-end"><FormattedNumber value={sumWithDeduction} --color="slategray" --font-weight="bold" /></td>
					</tr>
				</tbody>
			</Table>
		</Col>
	</Row>
</Container>
</div>
<div
	class="tab-pane fade"
	id="buying-tab-pane"
	role="tabpanel"
	aria-labelledby="buying-tab"
	tabindex="0">

	<Container>
		<Row>
			<Col>
				<h2>Inteckning</h2>
			</Col>
		</Row>

		<Row>
			<Col>
				<span class="label">
					Pantbrev
					<Icon id="icn-info-pawn" name="info-circle" style="info" />
				</span>
				<Popover target="icn-info-pawn" placement="right" title="Pantbrev" hideOnOutsideClick>
					<p>Summan av befintliga pantbrev på fastigheten.</p>
					<p>Banken behöver ha pantbrev för hela beloppet fastigheten är belånad på. Om det redan finns pantbrev på fastigheten tar du över dessa. Behöver du låna högre belopp än det finns pantbrev för så tecknas nya pantbrev.</p>
				</Popover>
			</Col>
		</Row>
		<Row>
			<Col>
				<Input
					id="inp-pawn"
					type="number"
					placeholder="Pantbrev"
					bind:value={pawn}
					min="0"
					max="{loan}"
					step="25000"
				/>
			</Col>
		</Row>
		<Row>
			<Col>
				<Input
					id="inp-pawn"
					type="range"
					bind:value={pawn}
					min="0"
					max="{loan}"
					step="25000"
				/>
			</Col>
		</Row>
		<Row>
			<Col>
				<Details id="pantbrev" label="Pantbrev" value={pantbrev}>
					<span slot="description">
						<p>Avgift att betala för nya pantbrev.</p>
						<p>Kostnaden för ett nytt pantbrev är 2% av beloppet du behöver låna - en så kallad stämpelskatt. Du behöver också betala 375 kronor i expedieringsavgift till Lantmäteriet.</p>
						{#if pantbrev > 0}
						<p>Eftersom bolånet är större än befintliga pantbrev är detta avgiften som ska betalas för nytecknade pantbrev.</p>
						{/if}
					</span>
				</Details>
			</Col>
			<Col>
				<Details id="lagfart" label="Lagfart" value={lagfart}>
					<span slot="description">
						<p>Avgift att betala för lagfarten.</p>
						<p>Kostnaden för ett lagfarten är 1,5% av fastighetens pris. Du behöver också betala 825 kronor i expedieringsavgift till Lantmäteriet.</p>
					</span>
				</Details>
			</Col>
		</Row>

		<Row>
			<Col>
				<h2>Försäljning</h2>
			</Col>
		</Row>

		<Row>
			<Col>
				<span class="label">
					Mäklararvode
					<Icon id="icn-info-realtor" name="info-circle" style="info" />
				</span>
				<Popover target="icn-info-realtor" placement="right" title="Mäklararvode" hideOnOutsideClick>
					<p>Mäklarens arvode vid försäljning av nuvarande bostad.</p>
					<p>Denna kostnad är avdragsgill i deklarationen efter en framtida försäljning av fastigheten, men det är en aktuell avgift som måste betalas <strong>nu</strong>. Avgiften dras av från försäljningen av fastigheten.</p>
				</Popover>
			</Col>
		</Row>
		<Row>
			<Col>
				<Input
					id="inp-realtor"
					type="number"
					placeholder="Adressändring"
					bind:value={realtorFee}
					min="0"
					max="200000"
					step="1000"
				/>
			</Col>
		</Row>
		<Row>
			<Col>
				<Input
					id="inp-realtor"
					type="range"
					bind:value={realtorFee}
					min="0"
					max="200000"
					step="1000"
				/>
			</Col>
		</Row>
		<Row>
			<Col>
				<span class="label">
					Annonsering
					<Icon id="icn-info-marketing" name="info-circle" style="info" />
				</span>
				<Popover target="icn-info-marketing" placement="right" title="Annonsering" hideOnOutsideClick>
					Avgift för annonsering vid försäljning av nuvarande bostad, ex. Hemnet.
				</Popover>
			</Col>
		</Row>
		<Row>
			<Col>
				<Input
					id="inp-marketing"
					type="number"
					placeholder="Annonsering"
					bind:value={marketing}
					min="0"
					max="20000"
					step="500"
				/>
			</Col>
		</Row>
		<Row>
			<Col>
				<Input
					id="inp-marketing"
					type="range"
					bind:value={marketing}
					min="0"
					max="20000"
					step="500"
				/>
			</Col>
		</Row>
		<Row>
			<Col>
				<span class="label">
					Styling
					<Icon id="icn-info-styling" name="info-circle" style="info" />
				</span>
				<Popover target="icn-info-styling" placement="right" title="Styling" hideOnOutsideClick>
					Avgift för home styling, ex. kostnaden för en stylingtjänst eller inköp av hemdekoration.
				</Popover>
			</Col>
		</Row>
		<Row>
			<Col>
				<Input
					id="inp-styling"
					type="number"
					placeholder="Styling"
					bind:value={stylingFee}
					min="0"
					max="20000"
					step="500"
				/>
			</Col>
		</Row>
		<Row>
			<Col>
				<Input
					id="inp-styling"
					type="range"
					bind:value={stylingFee}
					min="0"
					max="20000"
					step="500"
				/>
			</Col>
		</Row>
		<Row>
			<Col>
				<span class="label">
					Dubbelboende
					<Icon id="icn-info-double" name="info-circle" style="info" />
				</span>
				<Popover target="icn-info-double" placement="right" title="Dubbelboende" hideOnOutsideClick>
					Kostnad för eventuellt dubbelboende under en viss tid.
				</Popover>
			</Col>
		</Row>
		<Row>
			<Col>
				<Input
					id="inp-double"
					type="number"
					placeholder="Dubbelboende"
					bind:value={doubleLiving}
					min="0"
					max="500000"
					step="10000"
				/>
			</Col>
		</Row>
		<Row>
			<Col>
				<Input
					id="inp-double"
					type="range"
					bind:value={doubleLiving}
					min="0"
					max="500000"
					step="10000"
				/>
			</Col>
		</Row>

		<Row>
			<Col>
				<h2>Flytt</h2>
			</Col>
		</Row>

		<Row>
			<Col>
				<span class="label">
					Överlåtelseavgift
					<Icon id="icn-info-transfer" name="info-circle" style="info" />
				</span>
				<Popover target="icn-info-transfer" placement="right" title="Överlåtelseavgift" hideOnOutsideClick>
					<p>Vissa bostadsrättsföreningar tar ut en administrativ avgift för överlåtesen av bostaden.</p>
					<p>I vissa föreningar betalas den av <strong>säljaren</strong>, inte av köparen.</p>
				</Popover>
			</Col>
		</Row>
		<Row>
			<Col>
				<Input
					id="inp-transfer"
					type="number"
					placeholder="Överlåtelseavgift"
					bind:value={transferFee}
					min="0"
					max="2000"
					step="50"
				/>
			</Col>
		</Row>
		<Row>
			<Col>
				<Input
					id="inp-transfer"
					type="range"
					bind:value={transferFee}
					min="0"
					max="2000"
					step="50"
				/>
			</Col>
		</Row>
		<Row>
			<Col>
				<span class="label">
					Medlemsavgift
					<Icon id="icn-info-membership" name="info-circle" style="info" />
				</span>
				<Popover target="icn-info-membership" placement="right" title="Medlemsavgift" hideOnOutsideClick>
					Vissa bostadsrättsföreningar tar ut en avgift för registrering som medlem i föreningen.
				</Popover>
			</Col>
		</Row>
		<Row>
			<Col>
				<Input
					id="inp-membership"
					type="number"
					placeholder="Överlåtelseavgift"
					bind:value={membershipFee}
					min="0"
					max="2000"
					step="50"
				/>
			</Col>
		</Row>
		<Row>
			<Col>
				<Input
					id="inp-membership"
					type="range"
					bind:value={membershipFee}
					min="0"
					max="2000"
					step="50"
				/>
			</Col>
		</Row>
		<Row>
			<Col>
				<span class="label">
					Adressändring
					<Icon id="icn-info-address" name="info-circle" style="info" />
				</span>
				<Popover target="icn-info-address" placement="right" title="Adressändring" hideOnOutsideClick>
					Kostnaden för att göra en adressändring.
				</Popover>
			</Col>
		</Row>
		<Row>
			<Col>
				<Input
					id="inp-address"
					type="number"
					placeholder="Adressändring"
					bind:value={addressChange}
					min="0"
					max="1000"
					step="100"
				/>
			</Col>
		</Row>
		<Row>
			<Col>
				<Input
					id="inp-address"
					type="range"
					bind:value={addressChange}
					min="0"
					max="1000"
					step="100"
				/>
			</Col>
		</Row>
		<Row>
			<Col>
				<span class="label">
					Flyttlådor
					<Icon id="icn-info-boxes" name="info-circle" style="info" />
				</span>
				<Popover target="icn-info-boxes" placement="right" title="Flyttlådor" hideOnOutsideClick>
					Kostnaden för flyttlådor, emballage, förvaring etc.
				</Popover>
			</Col>
		</Row>
		<Row>
			<Col>
				<Input
					id="inp-boxes"
					type="number"
					placeholder="Flyttlådor"
					bind:value={movingBoxes}
					min="0"
					max="10000"
					step="500"
				/>
			</Col>
		</Row>
		<Row>
			<Col>
				<Input
					id="inp-boxes"
					type="range"
					bind:value={movingBoxes}
					min="0"
					max="10000"
					step="500"
				/>
			</Col>
		</Row>
		<Row>
			<Col>
				<span class="label">
					Flytthjälp
					<Icon id="icn-info-help" name="info-circle" style="info" />
				</span>
				<Popover target="icn-info-help" placement="right" title="Flytthjälp" hideOnOutsideClick>
					Kostnaden för flytthjälp, från vänner eller från flyttfirma.
				</Popover>
			</Col>
		</Row>
		<Row>
			<Col>
				<Input
					id="inp-help"
					type="number"
					placeholder="Flytthjälp"
					bind:value={movingHelp}
					min="0"
					max="30000"
					step="500"
				/>
			</Col>
		</Row>
		<Row>
			<Col>
				<Input
					id="inp-help"
					type="range"
					bind:value={movingHelp}
					min="0"
					max="30000"
					step="500"
				/>
			</Col>
		</Row>
		<Row>
			<Col>
				<span class="label">
					Flyttstädning
					<Icon id="icn-info-clean" name="info-circle" style="info" />
				</span>
				<Popover target="icn-info-clean" placement="right" title="Flyttstädning" hideOnOutsideClick>
					Kostnaden för flyttstädning, antingen på egen hand eller via firma. Ibland är detta inkluderat i kostnaden för flytthjälp.
				</Popover>
			</Col>
		</Row>
		<Row>
			<Col>
				<Input
					id="inp-clean"
					type="number"
					placeholder="Flyttstädning"
					bind:value={movingCleaning}
					min="0"
					max="30000"
					step="500"
				/>
			</Col>
		</Row>
		<Row>
			<Col>
				<Input
					id="inp-clean"
					type="range"
					bind:value={movingCleaning}
					min="0"
					max="30000"
					step="500"
				/>
			</Col>
		</Row>

		<Row>
			<Col>
				<h2>Summa</h2>
			</Col>
		</Row>

		<Row cols="3">
			<Col>
				<Table borderless hover>
					<tbody>
						<tr>
							<td>Pantbrev</td>
							<td class="text-end"><FormattedNumber value={pantbrev} /></td>
						</tr>
						<tr>
							<td>Lagfart</td>
							<td class="text-end"><FormattedNumber value={lagfart} /></td>
						</tr>
						<tr>
							<td>Försäljning</td>
							<td class="text-end"><FormattedNumber value={sumSelling} /></td>
						</tr>
						<tr>
							<td>Flyttkostnader</td>
							<td class="text-end"><FormattedNumber value={sumMoving} /></td>
						</tr>
						<tr>
							<td style="font-weight:bold;">Summa</td>
							<td class="text-end"><FormattedNumber value={sumBuying} --font-weight="bold" /></td
							>
						</tr>
					</tbody>
				</Table>
			</Col>
		</Row>
	</Container>
</div>
</div>

<style>
	h2 {
		font-size: 1.2em;
	}
	.header {
		position: sticky;
		top: 0;
		z-index: 999;
		padding: 8pt 10pt;
		margin: 0 0 10pt 0;
		background: #555;
		color: #f1f1f1;
	}
</style>
