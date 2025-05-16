<script>
	// Destructure incoming props with defaults.
	let {
		items,
		thead, // Function or fragment for rendering the table header.
		tableClass, // Additional class for the table.
		tbodyClass, // Additional class for the tbody.
		trow, // Function or fragment for rendering a table row.
		tfoot, // Function or fragment for rendering the table footer.
		rowHeight = 40, // Height (in pixels) for each data row.
		headerHeight = 40, // Height (in pixels) for the header row.
		// 'start' and 'end' are the indices of the first/last visible rows (inclusive).
		start = $bindable(0),
		end = $bindable(0)
	} = $props();

	console.log('rowHeight:', rowHeight);
	console.log('headerHeight:', headerHeight);
	console.log('Total items:', items.length);

	// For testing purposes, you can create a dummy array of 120 items:
	// let items = $state(Array.from({ length: 120 }, (_, i) => ({ id: i, data: `Item ${i}` })));

	// Virtualization state variables.
	// 'viewport' is a reference to the scroll container (<svelte-virtual-table-viewport>),
	// and 'viewportHeight' is its reactive height.
	let viewport = $state(null);
	let viewportHeight = $state(0);
	let scrollOffset = $state(0); // Fractional scroll remainder for smooth scrolling

	// Determine how many full data rows are visible  (excluding the header).
	// Math.max ensures no negative value when viewportHeight is small.
	// Alternate testing: try Math.ceil(viewportHeight / rowHeight) instead of Math.floor.
	// const visibleCount = $derived(Math.ceil(viewportHeight / rowHeight));
	const visibleCount = $derived(Math.floor(Math.max(0, viewportHeight - headerHeight) / rowHeight));

	// Get the visible rows from the items array.
	// Since 'end' is inclusive, slice from start to (end + 1).
	const visible = $derived(items.slice(start, end + 1));

	// Reactively update 'start' and 'end' when the viewport dimensions or item count change.
	$effect(() => {
		console.log('Enter effect');
		if (items.length > 0) {
			console.log('effect: items.length: ', items.length);
			// Ensure 'start' is clamped to leave room for visible rows.
			start = Math.min(start, Math.max(0, items.length - visibleCount));
			// Set 'end' so that exactly visibleCount rows are displayed (or as many remain).
			end = Math.min(start + visibleCount - 1, items.length - 1);
		} else {
			start = 0;
			end = 0;
		}
		console.log('Exit effect: start =', start, ', end =', end);
	});

	// Debug effect to log current virtualization state.
	$effect(() => {
		console.log('Updated Visible rows:', visible);
		console.log('Updated Visible Count:', visibleCount);
		console.log('Updated viewportHeight:', viewportHeight);
		console.log('Updated rowHeight:', rowHeight);
		console.log('Updated headerHeight:', headerHeight);
	});

	// Scroll handler recalculates/updates start, end, and scrollOffset based on the scroll position.
	function handleScroll() {
		console.log('HandleScroll Enter');
		const scrollTop = viewport.scrollTop;
		// Calculate the new start index
		const calcNewStart = Math.floor(scrollTop / rowHeight);
		// Clamp newStart so that there's room for visibleCount rows.
		const newStart = Math.min(calcNewStart, Math.max(0, items.length - visibleCount));
		// Calculate the new end index
		const calcNewEnd = newStart + visibleCount - 1;
		const newEnd = Math.min(calcNewEnd, items.length - 1);
		// Compute the fractional remainder for smooth scrolling.
		const remainder = Math.round(scrollTop - newStart * rowHeight); // TODO: Verify if Math.round is necessary.
		console.log('HandleScroll: scrollTop=', scrollTop, 'newStart=', newStart, 'newEnd=', newEnd);

		// Update reactive variables
		start = newStart;
		end = newEnd;
		scrollOffset = remainder;

		console.log('HandleScroll EXIT');
	}
</script>

<svelte-virtual-table-viewport
	bind:this={viewport}
	onscroll={handleScroll}
	bind:offsetHeight={viewportHeight}
>
	<table class="table {tableClass}">
		<!-- Render the table header.
             The parent may optionally supply a <colgroup> as part of the header snippet. -->
		{@render thead?.(headerHeight)}

		<!-- The tbody applies a translateY transform for smooth fractional scrolling.
             Spacer rows ensure the visible content is correctly positioned. -->
		<tbody
			class="tbody {tbodyClass}"
			style="transform: translateY(-{scrollOffset}px); --vt-row-height: {rowHeight}px;"
		>
			{#if start > 0}
				<!-- Top spacer: creates space for the items above the visible range -->
				<tr class="top">
					<td colspan="5" class="spacer-cell">
						<div style="height: {start * rowHeight}px;"></div>
					</td>
				</tr>
			{/if}

			{#each visible as item, index}
				<!-- Render each visible row.
                     The trow snippet should enforce a fixed height (rowHeight). -->
				{@render trow(item, index, rowHeight)}
			{/each}

			{#if end < items.length - 1}
				<!-- Bottom spacer: creates space for the remaining items below the visible range -->
				<tr class="bottom">
					<td colspan="5" class="spacer-cell">
						<div style="height: {(items.length - end - 1) * rowHeight}px;"></div>
					</td>
				</tr>
			{/if}
		</tbody>

		{@render tfoot?.()}
	</table>
</svelte-virtual-table-viewport>

<style>
	/* Global reset */
	*,
	*::before,
	*::after {
		box-sizing: border-box;
	}
	/* Scroll container styling */
	svelte-virtual-table-viewport {
		display: block;
		height: 100%;
		overflow-y: auto;
		border: 1px solid #ccc;
		margin: 0;
	}
	/* Table styling: fixed layout preserves column widths */
	table {
		width: 100%;
		border-collapse: collapse;
		table-layout: fixed;
	}

	/* Spacer cells: remove default padding, margin, and border. */
	.spacer-cell {
		padding: 0;
		margin: 0;
		border: none;
	}

	/* Alternate testing: Uncomment to use a CSS variable for row height scaling
    .row {
		height: var(--vt-row-height);
		line-height: var(--vt-row-height);
		font-size: calc(var(--vt-row-height) * 0.35);
    }
    */
</style>
