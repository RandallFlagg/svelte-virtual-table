<script>
	// Assume items is an array of your 120 items.
	// For testing purposes, you can create a dummy array:
	// let items = $state(Array.from({ length: 120 }, (_, i) => ({ id: i, data: `Item ${i}` })));
	let {
		items,
		thead,
		trow,
		tfoot,
		rowHeight = 40,
		// 'start' and 'end' represent the first and last visible row indices (inclusive)
		start = $bindable(0), //TODO: Fix the numbers
		end = $bindable(10) //TODO: Fix the numbers
		// ...props
	} = $props();

	console.log('LENGTH: ', items.length);

	// Virtualization state variables.
	// start and end determine which items are visible.
	// let start = 0;
	// let end = 10; // initial visible count
	let viewport = $state(null); // reference to the scroll container (the <svelte-virtual-table-viewport>)
	// let viewportHeight = 0;
	// Keep separate state variables like before, but use $state for reactivity
	let viewportHeight = $state(0);

	let scrollOffset = $state(0); // fractional remainder (for smooth scrolling)

	// Assume the header occupies the same height as a data row.
	// Assume that the header takes one row height.
	const headerHeight = rowHeight;

	// Compute the number of fully visible data rows (excluding the header).
	// Using Math.max ensures we don’t get negative values when viewportHeight is small.
	//TODO: Which should be used?
	// const visibleCount = $derived(Math.ceil(viewportHeight / rowHeight));
	const visibleCount = $derived(Math.floor(Math.max(0, viewportHeight - headerHeight) / rowHeight));
	// Derived computed visible items.
	// In Svelte, this reactive statement will update whenever start or end changes.
	// Compute the visible items slice.
	// Compute the "visible" rows. Since "end" now is the last visible index (inclusive),
	// we slice from start to end+1.
	// Compute the "visible" rows slice.
	// Since "end" is the last visible index (inclusive), we slice from start to end+1.
	const visible = $derived(items.slice(start, end + 1));

	// When the viewportHeight or visibleCount changes, update end if needed.
	// Update end when start or visibleCount changes
	// Whenever start or visibleCount changes, update end accordingly.
	// Reactively re-calc start and end (in case items array or viewport is updated).
	$effect(() => {
		console.log('Enter effect');
		//		if (start + visibleCount !== end) {
		//			console.log('IN IF');
		//		end = Math.min(start + visibleCount, items.length);
		//		}
		if (items.length > 0) {
			// Clamp start so there’s room for visibleCount rows.
			// Clamp start so there's room for a full block of visible rows.
			start = Math.min(start, Math.max(0, items.length - visibleCount));
			// Set end so that exactly visibleCount rows are displayed (or as many as remain).
			// Set end so that exactly visibleCount rows are displayed (or as many remain).
			end = Math.min(start + visibleCount - 1, items.length - 1);
		} else {
			start = 0;
			end = 0;
		}
		console.log('Exit effect');
	});

	// $effect(() => {
	// 	const currentStart = start;
	// 	const currentVisibleCount = visibleCount;
	// 	const currentEnd = end; // Current value of end before potential update

	// 	// For debugging, let's see the values before the condition:
	// 	console.log(
	// 		`Updating 'end': start=<span class="math-inline">\{currentStart\}, visibleCount\=</span>{currentVisibleCount}, currentEnd=${currentEnd}`
	// 	);

	// 	if (currentStart + currentVisibleCount !== currentEnd) {
	// 		const newEnd = Math.min(currentStart + currentVisibleCount, items.length);
	// 		console.log(
	// 			`Condition met: currentEnd (<span class="math-inline">\{currentEnd\}\) \!\=\= calculated \(</span>{currentStart + currentVisibleCount}). Setting end to ${newEnd}`
	// 		);
	// 		end = newEnd;
	// 	} else {
	// 		console.log(
	// 			`Condition NOT met: currentEnd (<span class="math-inline">\{currentEnd\}\) \=\=\= calculated \(</span>{currentStart + currentVisibleCount}). 'end' not changed.`
	// 		);
	// 	}
	// });

	$effect(() => {
		console.log('Visible rows:', visible);
		console.log('Visible Count:', visibleCount);
		console.log('viewportHeight:', viewportHeight);
	});

	// Scroll handler: calculates new start/end based on current scroll
	// Scroll handler recalculates start, end, and the scroll offset (for smooth transform)
	function handleScroll() {
		// if (!viewport) return; // Guard if viewport isn't bound yet
		console.log('HandleScroll Enter');
		const scrollTop = viewport.scrollTop;
		// Calculate new start index. (Floor the scrollTop divided by rowHeight.)
		const calcNewStart = Math.floor(scrollTop / rowHeight);
		// Clamp newStart so there’s always enough room for visibleCount rows:
		const newStart = Math.min(calcNewStart, Math.max(0, items.length - visibleCount));
		// Calculate new end index, ensuring we don't exceed items.length.
		const calcNewEnd = newStart + visibleCount - 1;
		const newEnd = Math.min(calcNewEnd, items.length - 1);
		// Instead of using Math.floor alone, store the fractional remainder
		const remainder = Math.round(scrollTop - newStart * rowHeight); //TODO: check if we need Math.round. Remove if not.

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
	<table class="table">
		<!-- Parent can provide <colgroup> via its own snippet, if needed -->
		<!-- The header (and optionally <colgroup>) is provided via the parent's snippet -->
		{@render thead?.()}

		<!-- No transform here; we use spacer rows to position the content -->
		<!-- The tbody now uses a transform so that even fractional scroll is visible.
             Spacer rows position the visible block correctly. -->
		<tbody class="tbody" style="transform: translateY(-{scrollOffset}px);">
			{#if start > 0}
				<!-- The top spacer: its height is calculated as the total height for the hidden items -->
				<!-- Top spacer row for hidden items before the visible range -->
				<!-- Top spacer: height based on number of hidden rows above the visible section -->
				<tr class="top">
					<td colspan="5" class="spacer-cell">
						<div style="height: {start * rowHeight}px;"></div>
						<!-- <div style="height: {viewport ? viewport.scrollTop : 0}px;"></div> -->
						<!-- <div style="height: {Math.round(viewport ? viewport.scrollTop : 0)}px;"></div> -->
					</td>
				</tr>
			{/if}

			{#each visible as item, index}
				<!-- Render each visible data row; ensure each row's height is exactly rowHeight -->
				<!-- Render each visible row ensuring its height= rowHeight -->
				<!-- Render each visible row. The row template should enforce a fixed height (rowHeight) -->
				<!-- Render each visible row. The trow snippet should enforce a fixed height (rowHeight). -->
				{@render trow(item, index, rowHeight)}
			{/each}

			{#if end < items.length - 1}
				<!-- The bottom spacer: calculates the height for all the items after the visible ones -->
				<!-- Bottom spacer row for remaining items -->
				<!-- Bottom spacer: height based on the remaining hidden items.
                     Since 'end' is inclusive, we subtract one more (items.length - end - 1).-->
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
	/* Table styling: fixed layout to preserve custom widths */
	/* Table styling: fixed layout ensures proper column widths */
	table {
		width: 100%;
		border-collapse: collapse;
		table-layout: fixed;
	}

	/* Data row styling */
	tbody tr {
		height: 40px; /* fixed height per row */ /* ensure this matches rowHeight */ /*TODO: This should be removed or be dynamically set. */
	}
	/* th, */ /*TODO: Should this be removed or just moved to  another place? This seems to have no effect.*/
	td {
		border: 1px solid #ccc;
		padding: 8px;
		text-align: left;
	}

	/* Spacer cells, minimal styling, letting the inner <div> set the correct height. */
	/* Spacer cells: no padding, border, or margin */
	.spacer-cell {
		padding: 0;
		margin: 0;
		border: none;
	}
</style>
