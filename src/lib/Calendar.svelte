<script lang="ts">
	import { createEventDispatcher } from 'svelte';

	type KeyArray = {
		[key: string]: string[];
	};

	const weekdays: KeyArray = {
		fr: ['Lun', 'Mar', 'Mer', 'Jeu', 'Ven', 'Sam', 'Dim'],
		en: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']
	};
	const months: KeyArray = {
		fr: ['Jan', 'Fév', 'Mar', 'Avr', 'Mai', 'Juin', 'Juil', 'Aoû', 'Sep', 'Oct', 'Nov', 'Déc'],
		en: ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec']
	};

	const dispatch = createEventDispatcher();
	/* Options */
	export let minDate: null | string = null;
	export let maxDate: null | string = null;
	export let disabledDates: string[] = [];
	export let mode: string = 'single'; // display, single, multiple
	export let selectedDate: null | string = null; // format: YYYY-MM-DD
	export let selectedDates: string[] = []; // format: [YYYY-MM-DD]
	export let maxSelectableDates = 2;
	export let locale = 'en';
	export let allowMonthPicker = true;
	export let allowYearPicker = true;

	const currentDate = new Date();
	let selectedMonth = currentDate.getMonth() + 1;
	let selectedYear = currentDate.getFullYear();
	let showMonthPicker = false;
	let showYearPicker = false;
	let yearIndex = 0;

	const getDaysInMonth = (year: number, month: number) => {
		return new Date(year, month, 0).getDate();
	};

	const prevMonth = () => {
		if (selectedMonth === 1) {
			selectedMonth = 12;
			selectedYear--;
		} else {
			selectedMonth--;
		}
		dispatch('clickOnNext', selectedMonth);
	};
	const nextMonth = () => {
		if (selectedMonth === 12) {
			selectedMonth = 1;
			selectedYear++;
		} else {
			selectedMonth++;
		}
		dispatch('clickOnNext', selectedMonth);
	};

	const checkGoPrev = (selectedYear: number, selectedMonth: number, minDate: string | null) => {
		if (!minDate) {
			return true;
		}
		const { month, year } = getDateValues(minDate);
		return selectedMonth > month || selectedYear > year;
	};
	const checkGoNext = (selectedYear: number, selectedMonth: number, maxDate: string | null) => {
		if (!maxDate) {
			return true;
		}
		const { month, year } = getDateValues(maxDate);
		return selectedMonth < month || selectedYear < year;
	};

	const isDateDisabled = (date: string) => {
		if (disabledDates.includes(date)) {
			return true;
		}
		if (minDate && date < minDate) {
			return true;
		}
		if (maxDate && date > maxDate) {
			return true;
		}
		return false;
	};
	const generateDays = (
		selectedYear: number,
		selectedMonth: number,
		minDate: null | string,
		maxDate: null | string
	) => {
		const nbOfDays = getDaysInMonth(selectedYear, selectedMonth);
		let days = [];
		for (let i = 1; i <= nbOfDays; i++) {
			days.push({
				day: i,
				month: selectedMonth,
				year: selectedYear,
				date: formatedDate(new Date(selectedYear, selectedMonth - 1, i)),
				disabled: isDateDisabled(formatedDate(new Date(selectedYear, selectedMonth - 1, i)))
			});
		}
		for (let i = 1; i <= 35 - nbOfDays; i++) {
			days.push({
				day: i,
				month: selectedMonth + 1,
				year: selectedYear,
				date: formatedDate(new Date(selectedYear, selectedMonth, i)),
				disabled: isDateDisabled(formatedDate(new Date(selectedYear, selectedMonth, i))),
				isNextMonth: true
			});
		}
		return days;
	};

	const handleClickOnDate = (date: string) => {
		dispatch('clickOnDate', date);
		if (mode === 'single') {
			if (selectedDate === date) {
				selectedDate = null;
			} else {
				selectedDate = date;
			}
		} else if (mode === 'multiple') {
			if (selectedDates.includes(date)) {
				selectedDates = [...selectedDates.filter((d) => d !== date)];
			} else {
				if (selectedDates.length < maxSelectableDates) {
					selectedDates = [...selectedDates, date];
				} else {
					selectedDates.shift();
					selectedDates = [...selectedDates, date];
				}
			}
		}
	};

	const formatedDate = (date: Date): string => {
		return `${date.getFullYear()}-${('0' + (date.getMonth() + 1)).slice(-2)}-${(
			'0' + date.getDate()
		).slice(-2)}`;
	};

	const getDateValues = (date: string) => {
		const [year, month, day] = date.split('-');
		return {
			year: parseInt(year),
			month: parseInt(month),
			day: parseInt(day)
		};
	};

	const checkIfSelected = (date: string, selectedDate: null | string, selectedDates: string[]) => {
		if (mode === 'single') {
			if (selectedDate) {
				return selectedDate && selectedDate === date;
			}
			return false;
		} else {
			return selectedDates.map((d) => d).includes(date);
		}
	};

	const resetValues = () => {
		selectedDate = null;
		selectedDates = [];
	};

	const pickMonth = (month: number) => {
		selectedMonth = month + 1;
		showMonthPicker = false;
	};
	const pickYear = (year: number) => {
		selectedYear = year;
		showYearPicker = false;
	};

	$: canGoPrev = checkGoPrev(selectedYear, selectedMonth, minDate);
	$: canGoNext = checkGoNext(selectedYear, selectedMonth, maxDate);
	$: currentDays = generateDays(selectedYear, selectedMonth, minDate, maxDate);
	$: mode, resetValues();
	$: pickableYears = [...Array(9).keys()].map((i) => selectedYear - 9 * -yearIndex + i - 4);
</script>

<div class="calendar">
	<div class="calendar_header">
		<button disabled={!canGoPrev} on:click={prevMonth}> prev </button>
		<div>
			<span
				on:click={() => {
					allowMonthPicker && (showMonthPicker = true);
				}}
			>
				{months[locale.toLowerCase()][selectedMonth - 1]}
			</span>
			<span
				on:click={() => {
					allowYearPicker && (showYearPicker = true);
				}}
			>
				{selectedYear}
			</span>
		</div>
		<button disabled={!canGoNext} on:click={nextMonth}>next</button>
	</div>
	<div class="calendar_content">
		<div class="calendar_content_weekdays">
			{#each weekdays[locale.toLowerCase()] as day}
				<div class="calendar_content_weekdays_item">
					<span>{day}</span>
				</div>
			{/each}
		</div>
		{#each currentDays as day}
			<div
				class="date text-unselectable {day.isNextMonth ? 'next-month' : ''} {day.disabled
					? 'disabled'
					: checkIfSelected(day.date, selectedDate, selectedDates)
					? 'selected'
					: ''}"
				on:click={() => !day.disabled && handleClickOnDate(day.date)}
			>
				<span>
					{day.day}
				</span>
			</div>
		{/each}
	</div>
	{#if showMonthPicker}
		<div id="month_picker">
			{#each months[locale.toLowerCase()] as month, i}
				<div
					class="calendar_months_item {i + 1 === selectedMonth ? 'selected' : ''}"
					on:click={() => {
						pickMonth(i);
					}}
				>
					<span>{month}</span>
				</div>
			{/each}
		</div>
	{/if}
	{#if showYearPicker}
		<div id="year_picker">
			<button
				on:click={() => {
					yearIndex--;
				}}>-</button
			>
			<div id="year_picker_years">
				{#each pickableYears as year}
					<div
						class="year_picker_item {year === selectedYear ? 'selected' : ''}"
						on:click={() => pickYear(year)}
					>
						<span>{year}</span>
					</div>
				{/each}
			</div>
			<button
				on:click={() => {
					yearIndex++;
				}}>+</button
			>
		</div>
	{/if}
</div>

<style>
	#month_picker {
		position: absolute;
		top: 0;
		background-color: white;
		width: 100%;
		height: 100%;
		display: grid;
		grid-template-columns: repeat(4, 1fr);
		gap: 5px;
	}
	.calendar_months_item {
		background-color: #f5f5f5;
		display: flex;
		justify-content: center;
		align-items: center;
	}
	.calendar_months_item.selected {
		background-color: #ccc;
	}
	#year_picker {
		position: absolute;
		top: 0;
		background-color: white;
		width: 100%;
		height: 100%;
		display: flex;
		align-items: center;
	}
	#year_picker_years {
		display: grid;
		flex: 1;
		grid-template-columns: repeat(3, 1fr);
		grid-template-rows: repeat(3, 1fr);
		height: 100%;
		gap: 5px;
	}
	.year_picker_item {
		background-color: #f5f5f5;
		display: flex;
		justify-content: center;
		align-items: center;
	}
	.year_picker_item.selected {
		background-color: #ccc;
	}
</style>
