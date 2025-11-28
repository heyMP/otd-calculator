<script lang="ts">
	import { onMount } from 'svelte';
	import { browser } from '$app/environment';

	type Mode = 'out the door' | 'selling';

	let mode: Mode = 'out the door';
	let otdPrice: number | undefined;
	let vehiclePrice: number | undefined;
	let taxRate: number | undefined;
	let docFee: number | undefined;
	let titleFee: number | undefined;
	let regFee: number | undefined;
	let shippingFee: number | undefined;
	let addOnsFee: number | undefined;
	let customFees: { name: string; amount: number | undefined }[] = [];

	const storageKey = 'otd-calculator-state';
	let hydrated = false;

	// Save state to localStorage whenever it changes
	$: if (browser && hydrated) {
		const state = {
			mode,
			otdPrice,
			vehiclePrice,
			taxRate,
			docFee,
			titleFee,
			regFee,
			shippingFee,
			addOnsFee,
			customFees
		};
		localStorage.setItem(storageKey, JSON.stringify(state));
	}

	// Load state from localStorage when the component mounts
	onMount(() => {
		const savedStateJSON = localStorage.getItem(storageKey);
		if (savedStateJSON) {
			try {
				const savedState = JSON.parse(savedStateJSON);
				
				// Handle migration from old mode names
				let savedMode = savedState.mode;
				if (savedMode === 'reverse') savedMode = 'out the door';
				if (savedMode === 'forward') savedMode = 'selling';

				mode = savedMode || 'out the door';
				otdPrice = savedState.otdPrice;
				vehiclePrice = savedState.vehiclePrice;
				taxRate = savedState.taxRate;
				docFee = savedState.docFee;
				titleFee = savedState.titleFee;
				regFee = savedState.regFee;
				shippingFee = savedState.shippingFee;
				addOnsFee = savedState.addOnsFee;
				customFees = savedState.customFees || [];
			} catch {
				localStorage.removeItem(storageKey);
				setDefaultValues();
			}
		} else {
			setDefaultValues();
		}
		hydrated = true;
	});

	function setDefaultValues() {
		mode = 'out the door';
		otdPrice = 35000;
		taxRate = 7;
		docFee = 499;
		titleFee = 150;
		regFee = 200;
		shippingFee = 0;
		addOnsFee = 0;
		customFees = [];
		// vehiclePrice is calculated, so not set here initially
	}

	$: totalFees =
		(docFee || 0) +
		(titleFee || 0) +
		(regFee || 0) +
		(shippingFee || 0) +
		(addOnsFee || 0) +
		(customFees.reduce((sum, fee) => sum + (fee.amount || 0), 0) || 0);

	// Main calculation logic
	$: {
		if (hydrated) {
			if (mode === 'out the door') {
				const calculatedVehiclePrice = ((otdPrice || 0) - totalFees) / (1 + (taxRate || 0) / 100);
				vehiclePrice = Math.round(calculatedVehiclePrice * 100) / 100;
			} else {
				const currentTaxAmount = (vehiclePrice || 0) * ((taxRate || 0) / 100);
				const calculatedOtdPrice = (vehiclePrice || 0) + currentTaxAmount + totalFees;
				otdPrice = Math.round(calculatedOtdPrice * 100) / 100;
			}
		}
	}

	$: taxAmount = (vehiclePrice || 0) * ((taxRate || 0) / 100);

	function formatCurrency(value: number, withPlus = false) {
		const formatted = new Intl.NumberFormat('en-US', {
			style: 'currency',
			currency: 'USD'
		}).format(value || 0);
		return withPlus && value > 0 ? `+ ${formatted}` : formatted;
	}

	function formatNumber(value: number) {
		return new Intl.NumberFormat('en-US', {
			minimumFractionDigits: 2,
			maximumFractionDigits: 2
		}).format(value || 0);
	}

	function addCustomFee() {
		customFees = [...customFees, { name: '', amount: undefined }];
	}

	function removeCustomFee(index: number) {
		customFees = customFees.filter((_, i) => i !== index);
	}

	function reset() {
		setDefaultValues();
	}

	function toggleMode() {
		mode = mode === 'out the door' ? 'selling' : 'out the door';
	}
</script>

<svelte:head>
	<title>OTD Reverse Calc</title>
	<meta name="description" content="Out The Door (OTD) Reverse Calculator" />
	<link rel="preconnect" href="https://fonts.googleapis.com" />
	<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
	<link
		href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap"
		rel="stylesheet"
	/>
</svelte:head>

<main>
	<div class="calculator-card">
		<header class="calculator-header">
			<div class="title">
				<svg width="24" height="24" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg"><path d="M19.5 2H4.5C3.39543 2 2.5 2.89543 2.5 4V20C2.5 21.1046 3.39543 22 4.5 22H19.5C20.6046 22 21.5 21.1046 21.5 20V4C21.5 2.89543 20.6046 2 19.5 2Z" stroke="#059669" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/><path d="M7.5 7H16.5" stroke="#059669" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/><path d="M7.5 12H16.5" stroke="#059669" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/><path d="M7.5 17H12.5" stroke="#059669" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/></svg>
				<h1>OTD Calc</h1>
			</div>
			<div class="controls">
				<div class="mode-switcher">
					<span>Out the Door</span>
					<button
						class="switch"
						class:selling={mode === 'selling'}
						on:click={toggleMode}
						aria-label="Toggle calculation mode"
					>
						<span class="handle" />
					</button>
					<span>Selling</span>
				</div>
				<button on:click={reset} class="reset-btn" aria-label="Reset form">
					<svg width="20" height="20" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg"><path d="M23 4V10H17" stroke="#6B7280" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/><path d="M1 20V14H7" stroke="#6B7280" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/><path d="M3.51 9.49001C4.79373 5.92305 8.52841 3.50155 12.5593 4.07293C16.5902 4.6443 19.8951 7.82098 20.4883 11.841C21.0815 15.861 18.6322 19.5765 15 20.49" stroke="#6B7280" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/><path d="M20.49 14.5C19.2063 18.067 15.4716 20.4885 11.4407 19.9171C7.4098 19.3457 4.10488 16.169 3.51172 12.149C2.91855 8.12902 5.36781 4.41353 9.00001 3.51001" stroke="#6B7280" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/></svg>
				</button>
			</div>
		</header>

		<div class="calculator-body">
			<div class="left-column">
				<section
					class="price-section"
					class:active-input={mode === 'out the door'}
					class:calculated-result={mode === 'selling'}
				>
					<label for="otdPrice">
						<svg width="24" height="24" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg"><path d="M12 2C6.47715 2 2 6.47715 2 12C2 17.5228 6.47715 22 12 22C17.5228 22 22 17.5228 22 12C22 6.47715 17.5228 2 12 2Z" stroke="#10B981" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/><path d="M12 16V12" stroke="#10B981" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/><path d="M12 8H12.01" stroke="#10B981" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/></svg>
						Target "Out The Door" Price
					</label>
					<div class="input-container">
						<span class="adornment">$</span>
						<input id="otdPrice" type="number" bind:value={otdPrice} placeholder="35,000" readonly={mode === 'selling'} />
					</div>
					<p class="description">This is the final amount you want to write the check for.</p>
				</section>

				<section
					class="price-section"
					class:active-input={mode === 'selling'}
					class:calculated-result={mode === 'out the door'}
				>
					<label for="vehiclePrice">
						Vehicle Selling Price
					</label>
					<div class="input-container">
						<span class="adornment">$</span>
						<input id="vehiclePrice" type="number" bind:value={vehiclePrice} placeholder="31,916.82" readonly={mode === 'out the door'} />
					</div>
					<p class="description">This is the negotiated price of the car before any fees or taxes.</p>
				</section>


				<section class="fees-section">
					<h2>DEALERSHIP FEES & TAXES</h2>
					<div class="fees-grid">
						<div class="input-group">
							<label for="taxRate">% SALES TAX RATE</label>
							<div class="input-container">
								<input id="taxRate" type="number" bind:value={taxRate} placeholder="7" />
							</div>
						</div>
						<div class="input-group">
							<label for="docFee">DOC FEE</label>
							<div class="input-container">
								<span class="adornment">$</span>
								<input id="docFee" type="number" bind:value={docFee} placeholder="499" />
							</div>
						</div>
						<div class="input-group">
							<label for="titleFee">TITLE FEE</label>
							<div class="input-container">
								<span class="adornment">$</span>
								<input id="titleFee" type="number" bind:value={titleFee} placeholder="150" />
							</div>
						</div>
						<div class="input-group">
							<label for="regFee">REGISTRATION</label>
							<div class="input-container">
								<span class="adornment">$</span>
								<input id="regFee" type="number" bind:value={regFee} placeholder="200" />
							</div>
						</div>
						<div class="input-group">
							<label for="shippingFee">SHIP / HANDLING</label>
							<div class="input-container">
								<span class="adornment">$</span>
								<input id="shippingFee" type="number" bind:value={shippingFee} placeholder="0" />
							</div>
						</div>
						<div class="input-group">
							<label for="addOnsFee">DEALER ADD-ONS</label>
							<div class="input-container">
								<span class="adornment">$</span>
								<input id="addOnsFee" type="number" bind:value={addOnsFee} placeholder="0" />
							</div>
						</div>
					</div>
				</section>

				<section class="custom-fees-section">
					<div class="custom-fees-header">
						<span>CUSTOM FEES (NITROGEN, PREP, ETC)</span>
						<button on:click={addCustomFee}>+ Add Fee</button>
					</div>
					<div class="custom-fees-list">
						{#if customFees.length === 0}
							<p class="no-fees">No custom fees added.</p>
						{/if}
						{#each customFees as fee, i}
							<div class="custom-fee-item">
								<input class="custom-fee-name" type="text" bind:value={fee.name} placeholder="Fee Name" />
								<div class="input-container">
									<span class="adornment">$</span>
									<input class="custom-fee-amount" type="number" bind:value={fee.amount} placeholder="0" />
								</div>
								<button on:click={() => removeCustomFee(i)} class="remove-fee-btn">&times;</button>
							</div>
						{/each}
					</div>
				</section>
			</div>

			<div class="right-column">
				<div class="target-price-card">
					{#if mode === 'out the door'}
						<p>TARGET VEHICLE PRICE</p>
						<h2>{formatCurrency(vehiclePrice)}</h2>
						<span>Offer this amount to hit your goal.</span>
					{:else}
						<p>FINAL OUT THE DOOR PRICE</p>
						<h2>{formatCurrency(otdPrice)}</h2>
						<span>This is your total estimated cost.</span>
					{/if}
				</div>
				<div class="breakdown">
					<div class="breakdown-item">
						<span>Vehicle Selling Price</span>
						<span>{formatNumber(vehiclePrice)}</span>
					</div>
					<div class="breakdown-item">
						<span>Sales Tax ({taxRate || 0}%)</span>
						<span class="positive">{formatCurrency(taxAmount, true)}</span>
					</div>
					<div class="breakdown-item">
						<span>Total Fees</span>
						<span class="positive">{formatCurrency(totalFees || 0, true)}</span>
					</div>
					<hr />
					<div class="breakdown-item total">
						<span>Target OTD Price</span>
						<span>{formatCurrency(otdPrice || 0)}</span>
					</div>
				</div>
			</div>
		</div>
	</div>
</main>

<style>
	:root {
		--bg-color: #f3f4f6;
		--card-bg: #ffffff;
		--border-color: #e5e7eb;
		--text-primary: #111827;
		--text-secondary: #6b7280;
		--primary-green: #059669;
		--light-green-bg: #ecfdf5;
		--red-accent: #ef4444;
		--font-family: 'Inter', sans-serif;
	}

	* {
		box-sizing: border-box;
	}

	body {
		margin: 0;
		background-color: var(--bg-color);
		font-family: var(--font-family);
		color: var(--text-primary);
	}

	main {
		display: flex;
		justify-content: center;
		align-items: flex-start;
		padding: 2rem;
		min-height: 100vh;
	}
	
	h1, h2 {
		margin: 0;
	}

	.calculator-card {
		background-color: var(--card-bg);
		border-radius: 1.5rem;
		box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.25);
		width: 100%;
		max-width: 1100px;
		overflow: hidden;
	}

	.calculator-header {
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding: 1.5rem 2rem;
		border-bottom: 1px solid var(--border-color);
	}

	.calculator-header .title {
		display: flex;
		align-items: center;
		gap: 0.75rem;
	}

	.calculator-header h1 {
		font-size: 1.25rem;
		font-weight: 600;
	}
	
	.controls {
		display: flex;
		align-items: center;
		gap: 1.5rem;
	}

	.mode-switcher {
		display: flex;
		align-items: center;
		gap: 0.5rem;
		font-size: 0.875rem;
		color: var(--text-secondary);
	}
	.mode-switcher .switch {
		background-color: var(--primary-green);
		border: none;
		border-radius: 99px;
		width: 44px;
		height: 24px;
		padding: 2px;
		cursor: pointer;
		position: relative;
		transition: background-color 0.2s;
	}
	.mode-switcher .switch .handle {
		display: block;
		width: 20px;
		height: 20px;
		background-color: white;
		border-radius: 50%;
		transition: transform 0.2s ease-in-out;
		transform: translateX(0);
	}
	.mode-switcher .switch.selling .handle {
		transform: translateX(20px);
	}


	.reset-btn {
		background: none;
		border: 1px solid var(--border-color);
		border-radius: 50%;
		width: 36px;
		height: 36px;
		cursor: pointer;
		display: flex;
		justify-content: center;
		align-items: center;
	}
	.reset-btn:hover {
		background-color: #f9fafb;
	}

	.calculator-body {
		display: grid;
		grid-template-columns: 1fr 1fr;
		gap: 2rem;
		padding: 2rem;
	}
	
	.left-column, .right-column {
		display: flex;
		flex-direction: column;
		gap: 2rem;
	}

	/* Common input styles */
	label {
		font-size: 0.875rem;
		font-weight: 500;
		color: var(--text-secondary);
		display: flex;
		align-items: center;
		gap: 0.5rem;
	}

	.input-container {
		position: relative;
		margin-top: 0.5rem;
	}

	.input-container .adornment {
		position: absolute;
		left: 0.75rem;
		top: 50%;
		transform: translateY(-50%);
		color: var(--text-secondary);
		pointer-events: none;
	}
	
	input[type="number"] {
		width: 100%;
		border: 1px solid var(--border-color);
		border-radius: 0.5rem;
		padding: 0.75rem;
		font-size: 1.25rem;
		font-weight: 600;
		outline: none;
		transition: border-color 0.2s;
	}
	input[type="number"]:focus {
		border-color: var(--primary-green);
	}
	.input-container input[type="number"] {
		padding-left: 2rem;
	}


	/* Price Section */
	.price-section {
		border: 1px solid transparent;
		border-radius: 1rem;
		padding: 1.5rem;
		transition: all 0.2s ease-in-out;
	}
	.price-section.active-input {
		background-color: var(--light-green-bg);
		border-color: var(--primary-green);
	}
	.price-section.calculated-result {
		background-color: #f9fafb;
		opacity: 0.7;
	}
	.price-section.calculated-result input {
		color: var(--text-secondary);
	}
	.price-section label {
		color: var(--primary-green);
		font-weight: 600;
	}
	.price-section.calculated-result label {
		color: var(--text-secondary);
	}

	.price-section input {
		font-size: 2rem;
		background: transparent;
		border-color: rgba(5, 150, 105, 0.3);
		color: var(--text-primary);
	}
	.price-section.calculated-result input {
		font-size: 1.5rem;
		border-color: transparent;
	}

	.price-section .description {
		font-size: 0.875rem;
		color: var(--text-secondary);
		margin: 0.5rem 0 0 0;
	}

	/* OTD Target Section */
	.otd-target-section {
		background-color: var(--light-green-bg);
		border: 1px solid var(--primary-green);
		border-radius: 1rem;
		padding: 1.5rem;
	}
	.otd-target-section label {
		color: var(--primary-green);
		font-weight: 600;
	}
	.otd-target-section input {
		font-size: 2rem;
		background: transparent;
		border-color: rgba(5, 150, 105, 0.3);
		color: var(--text-primary);
	}
	.otd-target-section .description {
		font-size: 0.875rem;
		color: var(--text-secondary);
		margin: 0.5rem 0 0 0;
	}

	/* Fees Section */
	.fees-section h2 {
		font-size: 0.75rem;
		font-weight: 600;
		color: var(--text-secondary);
		letter-spacing: 0.05em;
		padding-bottom: 0.75rem;
		border-bottom: 1px solid var(--border-color);
		margin-bottom: 1.5rem;
	}

	.fees-grid {
		display: grid;
		grid-template-columns: 1fr 1fr;
		gap: 1.5rem;
	}

	.fees-grid input[type="number"] {
		font-size: 1rem;
	}
	
	/* Custom Fees Section */
	.custom-fees-section {
		border: 1px solid var(--border-color);
		border-radius: 1rem;
		padding: 1.5rem;
	}
	.custom-fees-header {
		display: flex;
		justify-content: space-between;
		align-items: center;
		font-size: 0.875rem;
		font-weight: 500;
	}
	.custom-fees-header button {
		background: none;
		border: 1px solid var(--primary-green);
		color: var(--primary-green);
		border-radius: 99px;
		padding: 0.25rem 0.75rem;
		cursor: pointer;
		font-weight: 500;
	}
	.custom-fees-list {
		margin-top: 1rem;
	}
	.no-fees {
		text-align: center;
		color: var(--text-secondary);
		font-size: 0.875rem;
		padding: 1rem 0;
	}
	.custom-fee-item {
		display: flex;
		gap: 0.5rem;
		align-items: center;
		margin-bottom: 0.5rem;
	}
	.custom-fee-name {
		flex-grow: 1;
		border: 1px solid var(--border-color);
		border-radius: 0.5rem;
		padding: 0.5rem 0.75rem;
		font-size: 0.875rem;
	}
	.custom-fee-item .input-container {
		width: 120px;
		margin-top: 0;
	}
	.custom-fee-item input[type="number"] {
		font-size: 0.875rem;
		padding: 0.5rem 0.5rem 0.5rem 1.75rem;
	}
	.remove-fee-btn {
		border: none;
		background: none;
		font-size: 1.5rem;
		cursor: pointer;
		color: var(--text-secondary);
	}

	/* Right Column */
	.right-column {
		background-color: #f9fafb;
		border-radius: 1rem;
		padding: 2rem;
	}

	.target-price-card {
		text-align: center;
		border: 1px solid var(--border-color);
		border-radius: 1rem;
		padding: 1.5rem;
		background: linear-gradient(to top, #ffffff, #f9fafb);
	}
	.target-price-card p {
		font-size: 0.875rem;
		font-weight: 600;
		color: var(--text-secondary);
		margin: 0;
	}
	.target-price-card h2 {
		font-size: 2.5rem;
		font-weight: 700;
		margin: 0.5rem 0;
		color: var(--text-primary);
	}
	.target-price-card span {
		font-size: 0.875rem;
		color: var(--text-secondary);
	}

	.breakdown {
		font-size: 1rem;
	}
	.breakdown-item {
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding: 1rem 0;
	}
	.breakdown-item span:first-child {
		color: var(--text-secondary);
	}
	.breakdown-item span:last-child {
		font-weight: 600;
	}
	.breakdown .positive {
		color: var(--red-accent);
	}
	.breakdown hr {
		border: none;
		border-top: 1px solid var(--border-color);
		margin: 0.5rem 0;
	}
	.breakdown .total span {
		color: var(--text-primary);
		font-weight: 700;
		font-size: 1.125rem;
	}
	
	@media (max-width: 900px) {
		.calculator-body {
			grid-template-columns: 1fr;
		}
	}
</style>
