<script lang="ts">
	import Calendar from '$lib/Calendar.svelte';
	import '../styles/index.css';

	type Event = {
		title: string;
		data: string;
	};

	let modes = ['single', 'multiple', 'display'];
	let options = {
		locale: 'fr',
		minDate: null,
		maxDate: null,
		mode: 'single',
		maxSelectableDates: 2
	};
	let values = {
		selectedDate: null,
		selectedDates: []
	};
	let events: Event[] = [];

	const addEvent = (title: string, data: string = '') => {
		events = [...events, { title, data }];
	};
</script>

<div class="container">
	<div id="options">
		<h2>OPTIONS</h2>
		<div>
			{#each modes as mode}
				<label>
					<input type="radio" bind:group={options.mode} value={mode} />
					{mode}
				</label>
			{/each}
		</div>
		{#if options.mode === 'multiple'}
			<div>
				<label for="">Max date selectables</label>
				<input type="number" min="2" bind:value={options.maxSelectableDates} />
			</div>
		{/if}
		<div>
			<span>Locale</span>
			<select bind:value={options.locale}>
				<option value="fr">Français</option>
				<option value="en">English</option>
			</select>
		</div>
		<div>
			<span>Min Date</span>
			<input type="date" bind:value={options.minDate} />
		</div>
		<div>
			<span>Max Date</span>
			<input type="date" bind:value={options.maxDate} />
		</div>
	</div>
	<div id="values">
		<h2>VALUES</h2>
		<div>
			<span>Selected Date</span>
			<input type="date" disabled bind:value={values.selectedDate} />
		</div>
		<div>
			<span>Selected Dates</span>
			<ul>
				{#each values.selectedDates as date}
					<li><input type="date" disabled bind:value={date} /></li>
				{/each}
			</ul>
		</div>
	</div>
	<div style="border: 1px solid #ccc; border-radius: 5px;">
		<Calendar
			{...options}
			bind:selectedDate={values.selectedDate}
			bind:selectedDates={values.selectedDates}
			on:clickOnDate={(e) => {
				addEvent('ClickOnDate', e.detail);
			}}
			on:clickOnPrev={(e) => {
				addEvent('ClickOnPrev', `New month: ${e.detail}`);
			}}
			on:clickOnNext={(e) => {
				addEvent('ClickOnNext', `New month: ${e.detail}`);
			}}
		/>
	</div>
	<div id="events">
		<h2>
			EVENTS
			<button
				on:click={() => {
					events = [];
				}}
			>
				clear
			</button>
		</h2>
		<ul>
			{#each events as event}
				<li>
					<span>{event.title}</span>
					<span>{event.data}</span>
				</li>
			{/each}
		</ul>
	</div>
</div>

<style>
	.container {
		display: grid;
		grid-template-columns: repeat(3, 1fr);
		gap: 10px;
	}
	#options,
	#values,
	#events {
		border-radius: 5px;
		background-color: white;
		border: 1px solid #ccc;
		width: 100%;
		display: flex;
		flex-direction: column;
		gap: 10px;
	}
</style>
