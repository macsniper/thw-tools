<script lang="ts">
	import Input from '$lib/Input.svelte';
	import LinkButton from '$lib/LinkButton.svelte';
	import {
		calculateMatchingClothingSizeForTables,
		clothingNameToFriendlyName,
		getMissingMeasurements,
		humanMeasurementToFriendlyName,
		getMeasurementTolerance,
		humanGenderToFriendlyString
	} from '$lib/clothing/clothingUtils';
	import type {
		MatchingClothingSizeTable,
		HumanMeasurement,
		ClothingMeasurementImportance,
		HumanGender,
		ClothingName,
		ClothingSizes
	} from '$lib/clothing/clothing';
	import type { PageData } from './$types';
	import Select from '$lib/Select.svelte';

	export let data: PageData;

	// gender, height, chest, waist, hip
	let genderValue: HumanGender = 'M';
	let heightValue = '';
	let chestValue = '';
	let waistValue = '';
	let hipValue = '';
	let insideLegLengthValue = '';

	let calculatedSizes: MatchingClothingSizeTable[];
	let missingMeasurements: Map<ClothingName, HumanMeasurement[]> = new Map<
		ClothingName,
		HumanMeasurement[]
	>();

	function calculate(
		gender: HumanGender,
		height: number | undefined,
		chestCircumference: number | undefined,
		waistCircumference: number | undefined,
		hipCircumference: number | undefined,
		insideLegLength: number | undefined
	) {
		const inputData: Record<HumanMeasurement, number | undefined> = {
			height,
			chestCircumference,
			waistCircumference,
			hipCircumference,
			insideLegLength
		};

		missingMeasurements = getMissingMeasurements(data.tables, inputData);

		const sizes = calculateMatchingClothingSizeForTables(data.tables, gender, inputData);
		sizes.sort((a, b) => a.name.localeCompare(b.name));

		calculatedSizes = sizes;
	}

	$: calculate(
		genderValue,
		heightValue ? Number(heightValue) : undefined,
		chestValue ? Number(chestValue) : undefined,
		waistValue ? Number(waistValue) : undefined,
		hipValue ? Number(hipValue) : undefined,
		insideLegLengthValue ? Number(insideLegLengthValue) : undefined
	);

	function sizeToString(
		humanMeasurement: HumanMeasurement,
		matchingClothingSize: ClothingSizes | undefined,
		measurementImportance: ClothingMeasurementImportance[]
	) {
		const friendlyName = humanMeasurementToFriendlyName(humanMeasurement);
		if (!matchingClothingSize) return `${friendlyName}: -`;

		const sizeValue = matchingClothingSize[humanMeasurement];
		if (!sizeValue) return `${friendlyName}: -`;

		const isToleranceAllowed = isToleranceAllowedInMeasurement(
			humanMeasurement,
			measurementImportance
		);

		if (isToleranceAllowed === false) {
			return `${friendlyName}: ${sizeValue.min} - ${sizeValue.max} cm (genau)`;
		}

		return `${friendlyName}: ${sizeValue.min} - ${sizeValue.max} cm`;
	}

	function isNessaryMeasurement(
		humanMeasurement: HumanMeasurement,
		measurementImportance: ClothingMeasurementImportance[]
	): boolean | undefined {
		const importance = measurementImportance.find(
			(importance) => importance.measurement === humanMeasurement
		);

		return importance !== undefined;
	}

	function isToleranceAllowedInMeasurement(
		humanMeasurement: HumanMeasurement,
		measurementImportance: ClothingMeasurementImportance[]
	): boolean | undefined {
		const importance = measurementImportance.find(
			(importance) => importance.measurement === humanMeasurement
		);
		if (!importance) return undefined;

		return importance.allowTolerance;
	}
</script>

<div class="p-4 flex flex-col gap-4">
	<span class="text-xl"
		>Hier kannst du die passende Größe für den neuen Einsatzanzug (MEA) finden. Gebe deine
		Körpermaße und dein Geschlecht ein.</span
	>

	<div class="flex flex-row flex-wrap gap-2">
		<Select
			options={[
				{ value: 'M', label: humanGenderToFriendlyString('M') },
				{ value: 'W', label: humanGenderToFriendlyString('W') }
			]}
			bind:selected={genderValue}
			label="Geschlecht"
		/>
		<Input
			bind:inputValue={heightValue}
			type="number"
			label={humanMeasurementToFriendlyName('height')}
			placeholder={`${humanMeasurementToFriendlyName('height')} in cm`}
		/>
		<Input
			bind:inputValue={chestValue}
			type="number"
			label={humanMeasurementToFriendlyName('chestCircumference')}
			placeholder={`${humanMeasurementToFriendlyName('chestCircumference')} in cm`}
		/>
		<Input
			bind:inputValue={waistValue}
			type="number"
			label={humanMeasurementToFriendlyName('waistCircumference')}
			placeholder={`${humanMeasurementToFriendlyName('waistCircumference')} in cm`}
		/>
		<Input
			bind:inputValue={hipValue}
			type="number"
			label={humanMeasurementToFriendlyName('hipCircumference')}
			placeholder={`${humanMeasurementToFriendlyName('hipCircumference')} in cm`}
		/>
		<Input
			bind:inputValue={insideLegLengthValue}
			type="number"
			label={humanMeasurementToFriendlyName('insideLegLength')}
			placeholder={`${humanMeasurementToFriendlyName('insideLegLength')} in cm`}
		/>
	</div>

	<div class="flex flex-col gap-2">
		{#if calculatedSizes}
			<div class="font-bold">Es wurden folgende Größen gefunden:</div>
			<div class="grid grid-flow-row-dense grid-cols-1 md:grid-cols-3 gap-2">
				{#each calculatedSizes as size (size.name)}
					<div class="flex flex-col justify-between gap-1 border-2 p-2 border-thw rounded-md">
						<div>
							<h2 class="text-xl font-bold">{clothingNameToFriendlyName(size.name)}:</h2>
							{#if size.matchingClothingSize}
								<div class="flex flex-col gap-2">
									<div>
										<p>Konfektionsgröße: {size.matchingClothingSize.size}</p>
										<p
											class={isNessaryMeasurement('height', size.measurementImportance)
												? 'font-bold'
												: ''}
										>
											{sizeToString(
												'height',
												size.matchingClothingSize,
												size.measurementImportance
											)}
										</p>
										<p
											class={isNessaryMeasurement('chestCircumference', size.measurementImportance)
												? 'font-bold'
												: ''}
										>
											{sizeToString(
												'chestCircumference',
												size.matchingClothingSize,
												size.measurementImportance
											)}
										</p>
										<p
											class={isNessaryMeasurement('waistCircumference', size.measurementImportance)
												? 'font-bold'
												: ''}
										>
											{sizeToString(
												'waistCircumference',
												size.matchingClothingSize,
												size.measurementImportance
											)}
										</p>
										<p
											class={isNessaryMeasurement('hipCircumference', size.measurementImportance)
												? 'font-bold'
												: ''}
										>
											{sizeToString(
												'hipCircumference',
												size.matchingClothingSize,
												size.measurementImportance
											)}
										</p>
										<p
											class={isNessaryMeasurement('insideLegLength', size.measurementImportance)
												? 'font-bold'
												: ''}
										>
											{sizeToString(
												'insideLegLength',
												size.matchingClothingSize,
												size.measurementImportance
											)}
										</p>
									</div>
									<div class="italic text-sm">
										[Toleranzfaktor: {getMeasurementTolerance(size.name)}]
									</div>
								</div>
							{:else if missingMeasurements.has(size.name)}
								<p class="font-bold text-red-500">Fehlende Maße:</p>
								<ul>
									{#each missingMeasurements.get(size.name) ?? [] as missingMeasurement}
										<li>
											{humanMeasurementToFriendlyName(missingMeasurement)}
											{#if isToleranceAllowedInMeasurement(missingMeasurement, size.measurementImportance) === false}
												(genau)
											{/if}
										</li>
									{/each}
								</ul>
							{:else}
								<p>No matching sizes found.</p>
							{/if}
						</div>
						<div>
							<LinkButton url={`${size.name}/${size.gender}/`} secondary
								>Maßtabelle ansehen</LinkButton
							>
						</div>
					</div>
				{/each}
			</div>
			<div>
				<div>
					<span class="font-bold">Info:</span> Die fettgedruckten Maße im Ergebnis eines Kleidungsstücks
					sind zwingend notwendig für die Größenberechnung.
				</div>
				<div>
					Fettgedruckte Maße ohne die Information "genau" haben je nach Kleidungsstück eine
					Toleranzfaktor.
				</div>
				<div>
					Die anderen (nicht dickgedruckten) Maße fließen nicht in die Berechnung der Größe ein.
				</div>
			</div>
		{:else}
			<p>No results yet.</p>
		{/if}
	</div>

	<img src="/clothing/measurement.png" alt="Größentabelle" class="w-full md:w-1/2 mx-auto" />
</div>