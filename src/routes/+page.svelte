<script lang="ts">
	import { onMount } from 'svelte';
	import {
		Button,
		Table,
		TableBody,
		TableBodyCell,
		TableBodyRow,
		TableHead,
		TableHeadCell,
		Checkbox,
		Pagination,
		PaginationItem,
		ChevronLeft,
		ChevronRight,
		ArrowKeyUp,
		ArrowKeyDown,
		TableSearch,
	} from 'flowbite-svelte';
	import dayjs from 'dayjs';
	import relativeTime from 'dayjs/plugin/relativeTime';
	import { page } from '$app/stores';
	import type { LinkType } from 'flowbite-svelte/types';

	dayjs.extend(relativeTime);

	interface Row {
		username: string;
		frequency: number;
		verifiedAt: string;
		updatedAt: string;
		definition: string[];
	}

	const debounce = (value: string) => {
		clearTimeout(timer);

		timer = setTimeout(() => {
			searchTerm = value;
		}, 500);
	};

	let timer: NodeJS.Timeout;
	let data: Row[] = [];
	let currentPage = 0;
	let totalPages = 0;
	let pageSize = 10;
	let searchTerm = '';
	let searchTermDebounced = '';
	let sortColumn = 'frequency';
	let sortDirection = 'desc';
	let token = '';
	let pages: LinkType[] = [];

	$: {
		pages = [
			{ name: '1', href: `/?page=1` },
			{ name: (currentPage + 2).toString(), href: `/?page=${currentPage + 2}` },
			{ name: (currentPage + 3).toString(), href: `/?page=${currentPage + 3}` },
			{ name: (currentPage + 4).toString(), href: `/?page=${currentPage + 4}` },
			{ name: totalPages.toString(), href: `/?page=${totalPages}` },
		];
	}

	$: {
		const pageNumber = $page.url.searchParams.get('page');

		if (pageNumber) {
			currentPage = parseInt(pageNumber) - 1;
		}
	}

	$: debounce(searchTermDebounced);

	onMount(async () => {
		token = $page.url.searchParams.get('token') ?? 'none';

		fetchData(currentPage, pageSize, searchTerm, sortColumn, sortDirection);
	});

	function handleSort(column: any) {
		if (sortColumn === column) {
			sortDirection = sortDirection === 'asc' ? 'desc' : 'asc';
		} else {
			sortColumn = column;
			sortDirection = 'asc';
		}
	}

	$: fetchData(currentPage, pageSize, searchTerm, sortColumn, sortDirection);

	async function fetchData(
		currentPage: number,
		pageSize: number,
		searchTerm: string,
		sortColumn: string,
		sortDirection: string
	) {
		const response = await fetch(
			`https://api.matteopolak.com/names?token=${token}&offset=${
				currentPage * pageSize
			}&limit=${pageSize}${
				searchTerm ? `&search=${searchTerm}` : ''
			}&column=${sortColumn}&sort=${sortDirection}`
		);

		const json = await response.json();
		data = json.results;
		totalPages = Math.ceil(json.total / pageSize);
	}
</script>

<TableSearch
	placeholder="Search by username"
	hoverable={true}
	bind:inputValue={searchTermDebounced}
	bind:currentPage
>
	<TableHead>
		<TableHeadCell>
			<button on:click={() => handleSort('length')}>
				USERNAME

				{#if sortColumn === 'length'}
					{#if sortDirection === 'asc'}
						<ArrowKeyUp class="inline w-4 h-4 ml-1" />
					{:else}
						<ArrowKeyDown class="inline w-4 h-4 ml-1" />
					{/if}
				{/if}
			</button>
		</TableHeadCell>
		<TableHeadCell>
			<button on:click={() => handleSort('frequency')}>
				FREQUENCY

				{#if sortColumn === 'frequency'}
					{#if sortDirection === 'asc'}
						<ArrowKeyUp class="inline w-4 h-4 ml-1" />
					{:else}
						<ArrowKeyDown class="inline w-4 h-4 ml-1" />
					{/if}
				{/if}
			</button>
		</TableHeadCell>
		<TableHeadCell>
			<button on:click={() => handleSort('verifiedAt')}>
				CHECKED

				{#if sortColumn === 'verifiedAt'}
					{#if sortDirection === 'asc'}
						<ArrowKeyUp class="inline w-4 h-4 ml-1" />
					{:else}
						<ArrowKeyDown class="inline w-4 h-4 ml-1" />
					{/if}
				{/if}
			</button>
		</TableHeadCell>
		<TableHeadCell>
			<button on:click={() => handleSort('updatedAt')}>
				UPDATED

				{#if sortColumn === 'updatedAt'}
					{#if sortDirection === 'asc'}
						<ArrowKeyUp class="inline w-4 h-4 ml-1" />
					{:else}
						<ArrowKeyDown class="inline w-4 h-4 ml-1" />
					{/if}
				{/if}
			</button>
		</TableHeadCell>
	</TableHead>
	<TableBody tableBodyClass="divide-y">
		{#each data as row}
			<TableBodyRow>
				<TableBodyCell>{row.username}</TableBodyCell>
				<TableBodyCell>{row.frequency.toFixed(3)}</TableBodyCell>
				<TableBodyCell>{dayjs(row.verifiedAt).fromNow()}</TableBodyCell>
				<TableBodyCell>{dayjs(row.updatedAt).fromNow()}</TableBodyCell>
			</TableBodyRow>
		{/each}
		{#if data.length === 0}
			<TableBodyRow>
				<TableBodyCell colspan={5}>No results found</TableBodyCell>
			</TableBodyRow>
		{/if}
	</TableBody>
</TableSearch>

<div class="flex justify-center mt-2">
	<Pagination
		{pages}
		on:previous={() => {
			if (currentPage > 0) currentPage--;
		}}
		on:next={() => {
			if (currentPage < totalPages - 1) currentPage++;
		}}
		icon
	>
		<svelte:fragment slot="prev">
			<span class="sr-only">Previous</span>
			<ChevronLeft class="w-5 h-5" />
		</svelte:fragment>
		<svelte:fragment slot="next">
			<span class="sr-only">Next</span>
			<ChevronRight class="w-5 h-5" />
		</svelte:fragment>
	</Pagination>
</div>
