<script lang="ts">
	import { onMount } from 'svelte';
	import {
		Badge,
		TableBody,
		TableBodyCell,
		TableBodyRow,
		TableHead,
		TableHeadCell,
		ArrowKeyUp,
		ArrowKeyDown,
	} from 'flowbite-svelte';
	import TableSearch from '../components/TableSearch.svelte';
	import Time from 'svelte-time';
	import { page } from '$app/stores';
	import type { LinkType } from 'flowbite-svelte/types';

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
		}, 250);
	};

	let timer: NodeJS.Timeout;
	let data: Row[] = [];
	let currentPage = 0;
	let totalPages = 1;
	let pageSize = 10;
	let searchTerm = '';
	let searchTermDebounced = '';
	let sortColumn = 'frequency';
	let sortDirection = 'desc';
	let token = '';
	let pages: LinkType[] = [];

	$: {
		pages = [
			{ name: '1', href: `/?token=${token}&page=1` },
			{
				name: (currentPage + 2).toString(),
				href: `/?token=${token}&page=${currentPage + 2}`,
			},
			{
				name: (currentPage + 3).toString(),
				href: `/?token=${token}&page=${currentPage + 3}`,
			},
			{
				name: (currentPage + 4).toString(),
				href: `/?token=${token}&page=${currentPage + 4}`,
			},
			{
				name: totalPages.toString(),
				href: `/?token=${token}&page=${totalPages}`,
			},
		];
	}

	$: {
		const pageNumber = $page.url.searchParams.get('page');

		if (pageNumber) {
			$page.url.searchParams.delete('page');
			currentPage = Math.max(0, Math.min(parseInt(pageNumber), totalPages) - 1);
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

		currentPage = 0;
	}

	$: {
		if (token) {
			fetchData(currentPage, pageSize, searchTerm, sortColumn, sortDirection);
		}
	}

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

		console.log(response.status);

		if (response.status !== 200) {
			data = [];
			totalPages = 1;

			return;
		}

		const json = await response.json();

		data = json.results;
		totalPages = Math.ceil(json.total / pageSize);
	}
</script>

<TableSearch
	placeholder="Search by username"
	hoverable={true}
	striped={true}
	bind:inputValue={searchTermDebounced}
	bind:currentPage
	bind:pageSize
	{pages}
	{totalPages}
>
	<caption
		class="p-5 text-lg font-semibold text-left text-gray-900 bg-white dark:text-white dark:bg-gray-800"
	>
		Minecraft Name Index
		<p class="mt-1 text-sm font-normal text-gray-500 dark:text-gray-400">
			Browse a list of available Minecraft usernames and view their frequency in
			the English language.
		</p>
	</caption>
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
				<TableBodyCell>
					{row.username}
					<span style="float: right">
						{#if row.username.length <= 7}
							<Badge color="purple">Short</Badge>
						{/if}
						{#if row.frequency >= 0.5}
							<Badge color="pink">Common</Badge>
						{/if}
						{#if new Date(row.updatedAt).getTime() > Date.now() - 86_400_000}
							<Badge color="green">New</Badge>
						{/if}
					</span>
				</TableBodyCell>
				<TableBodyCell>
					{row.frequency.toFixed(3)}
				</TableBodyCell>
				<TableBodyCell
					><Time live relative timestamp={row.verifiedAt} /></TableBodyCell
				>
				<TableBodyCell>
					<Time live relative timestamp={row.updatedAt} />
				</TableBodyCell>
			</TableBodyRow>
		{/each}
		{#if data.length === 0}
			<TableBodyRow>
				<TableBodyCell colspan={5}>No results found</TableBodyCell>
			</TableBodyRow>
		{/if}
	</TableBody>
</TableSearch>
