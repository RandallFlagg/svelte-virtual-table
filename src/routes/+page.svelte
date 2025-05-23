<script>
	import { onMount } from 'svelte';
	import VirtualTable from '../lib/index';
	// import VirtualTable from 'svelte-virtual-table'

	let items = $state([]);
	let searchTerm = $state('');
	let start = $state(0);
	let start2 = $state(0);
	let end = $state(10);
	let end2 = $state(10);

	let dataPromise = $state(null);
	async function getData() {
		let cachedData = localStorage.getItem('hnData');

		if (cachedData) {
			items = JSON.parse(cachedData);
		} else {
			let dataItems = [];
			for (let page = 1; page < 5; page++) {
				let res = await fetch(`https://node-hnapi.herokuapp.com/news?page=${page}`);
				let data = await res.json();
				dataItems = dataItems.concat(data);
				//dataItems = [...dataItems, ...data];
			}
			items = dataItems;
			localStorage.setItem('hnData', JSON.stringify(items)); // Cache for next reload
		}

		return items;
	}

	onMount(() => {
		dataPromise = getData(); // This ensures it updates reactively and correctly
	});

	let filteredList = $derived(
		items.filter((item) => item.title.toUpperCase().includes(searchTerm.toUpperCase()))
	);
</script>

<h1>Virtual Table Test</h1>

<p>
	This is an example-project for the Svelte Virtual (Sortable) Table component. Checkout the source
	on <a href="https://github.com/BernhardWebstudio/svelte-virtual-table">GitHub</a>. The table's
	example content is loaded from HackerNews — all rights reserved. The author has no association
	with Svelte or Hacker News and is not liable and/or otherwise responsible for any of the contents
	in these tables.
</p>

Filter:
<input bind:value={searchTerm} id="searchTerm" />

{#await dataPromise}
	Loading...
{:then}
	<p>Loaded {filteredList.length} items.</p>
	<h3>Without border-collapse:</h3>
	<p>Start: {start}, end: {end}</p>
	<div class="virtual-table-demo">
		<VirtualTable
			items={filteredList}
			className="test1 test2"
			rowHeight="40"
			headerHeight="40"
			tableClass="collapse"
			tbodyClass="tbodyStyle"
			bind:start
			bind:end
		>
			{#snippet thead(headerHeight)}
				<!-- <colgroup>
					<col class="col" />
					<col class="col" />
					<col class="col" />
					<col class="col" />
					<col class="col" />
				</colgroup> -->
				<thead class="sticky-header">
					<tr>
						<th data-sort="title" style="height: {headerHeight}px; line-height: {headerHeight}px;"
							>Title</th
						>
						<th data-sort="user" style="height: {headerHeight}px; line-height: {headerHeight}px;">User</th
						>
						<th data-sort="domain" style="height: {headerHeight}px; line-height: {headerHeight}px;"
							>Domain</th
						>
						<th
							data-sort="time"
							data-sort-initial="descending"
							style="height: {headerHeight}px; line-height: {headerHeight}px;">Time ago</th
						>
						<th
							data-sort="comments_count"
							style="height: {headerHeight}px; line-height: {headerHeight}px;">Comments</th
						>
					</tr>
				</thead>
			{/snippet}
			{#snippet trow(item, index, rowHeight)}
				<tr class="tr" style="height: {rowHeight}px;">
					<td class="td"><a href={item.url} target="_blank">{item.title}</a></td>
					<td class="td">{item.user}</td>
					<td class="td">{item.domain}</td>
					<td class="td">{item.time_ago}</td>
					<td class="td">{item.comments_count}</td>
				</tr>
			{/snippet}
			{#snippet tfoot()}
				<!-- Add TEST-->
			{/snippet}
		</VirtualTable>
	</div>

	<h3>With border-collapse:</h3>
	<p>Start: {start2}, end: {end2}</p>
	<div class="virtual-table-demo">
		<VirtualTable
			items={filteredList}
			class="test1 test2"
			requireBorderCollapse="true"
			bind:start={start2}
			bind:end={end2}
		>
			{#snippet thead()}
				<tr>
					<th data-sort="title">Title</th>
					<th data-sort="user">User</th>
					<th data-sort="domain">Domain</th>
					<th data-sort="time" data-sort-initial="descending">Time ago</th>
					<th data-sort="comments_count">Comments</th>
				</tr>
			{/snippet}
			{#snippet trow(item)}
				<tr class="tr">
					<td class="td"><a href={item.url} target="_blank">{item.title}</a></td>
					<td class="td">{item.user}</td>
					<td class="td">{item.domain}</td>
					<td class="td">{item.time_ago}</td>
					<td class="td">{item.comments_count}</td>
				</tr>
			{/snippet}
		</VirtualTable>
	</div>
{:catch error}
	<p style="color: red">{error.message}</p>
{/await}

<p>
	Thanks for checking out this Demo. Refer to the <a
		href="https://github.com/BernhardWebstudio/svelte-virtual-table#readme">README</a
	>
	of the original project on how to use this component and the
	<a href="https://github.com/BernhardWebstudio/svelte-virtual-table/issues">GitHub-project</a> in general
	for known issues.
</p>

<style>
	/* :global(html),
	:global(body) {
		height: 100%;
		margin: 0;
		padding: 0;
	} */

	/* TODO: This should be passed to the table in the component */
	.collapse {
		border-collapse: collapse;
	}

	/* TODO: This should be passed to the tbody in the component */
	.tbodyStyle {
		background-color: blue;
	}

	.virtual-table-demo {
		/* display: flex;
		flex-direction: column; */
		/* Define an explicit height; you can adjust this value as needed */
		height: 520px; /*This should be a multiplaction of the rowheight in px */
		/* Alternatively, set both height and max-height if you want a cap: */
		/* max-height: 50vh; */
		/* Remove margins if they interfere with available space */
		/* margin: 20px 0; */
	}

	/* .col{
		width: 100px;
	} */

	.sticky-header {
		background: #f5f5f5;
		border: none;
		box-shadow: inset 0 -1px 0 0 #ccc;
		height: 40px; /* or use rowHeight */
		position: sticky;
		top: 0;
		z-index: 10;
	}

	/* Header styles – parent can also add class="sticky-header" in its own snippet */
	/* Use a box-shadow to simulate a consistent vertical border */
	/* Height is the same as rowHeight. TODO: Check if this can be removed. */
	thead th {
		border: none;
		box-shadow:
			inset 1px 0 0 0 #ccc,
			inset -1px 0 0 0 #ccc;
		font-size: 14px;
		/* height: 40px; 
		line-height: 40px; */
		padding: 0 8px;
		text-align: left;
	}

	/* :global(body) {
		background-color: white;
	} */

	/* :global(table) {
		min-height: min(10rem, 50vh);
	} */

	/* :global(.test1) {
		width: 100%;
		border: 1px solid black;
	}

	:global(.test1 thead) {
		text-align: left;
		border-bottom: 10px solid black;
	} */

	/**
  * This border is only visible when 
  */
	/* :global(.test1 tr) {
		border-bottom: 10px solid grey;
		border-top: 10px solid grey;
	}

	:global(.test1 td:not(:last-of-type)) {
		border-right: 1px solid grey;
	}
	:global(.test1 th:not(:last-of-type)) {
		border-right: 1px solid grey;
	} */

	/**
  * Specify fixed widths
  */
	/* .td:first-of-type,
	td:first-of-type,
	:global(.test1 th:first-of-type) {
		width: 45vw;
	}

	td,
	.td,
	:global(.test1 th) {
		width: calc((45vw - 10px) / 4);
		word-wrap: break-word;
	}

	:global(.test1 th:last-of-type),
	.td:last-of-type,
	td:last-of-type {
		text-align: right;
	} */
</style>
