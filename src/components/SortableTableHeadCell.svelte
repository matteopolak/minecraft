<script lang="ts">
	import { ArrowKeyDown, ArrowKeyUp, TableHeadCell } from 'flowbite-svelte';
	export let handleSort: (column: string) => void;
	export let sortColumn = '';
	export let sortDirection = '';
	export let column = '';
</script>

<TableHeadCell>
	<button on:click={() => handleSort(column)}>
		<slot />

		{#if sortColumn === column}
			{#if sortDirection === 'asc'}
				<ArrowKeyUp class="inline w-4 h-4 ml-1" />
			{:else}
				<ArrowKeyDown class="inline w-4 h-4 ml-1 mb-1" />
			{/if}
		{/if}
	</button>
</TableHeadCell>
