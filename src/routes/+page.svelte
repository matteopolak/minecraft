<script lang="ts">
	import { onMount } from 'svelte';
	import {
		Badge,
		TableBody,
		TableBodyCell,
		TableBodyRow,
		TableHead,
		Popover,
		InformationCircle,
	} from 'flowbite-svelte';
	import SortableTableHeadCell from '../components/SortableTableHeadCell.svelte';
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
		// show up to 5 pages, with the first and last page separated by ellipsis buttons
		// if the current page is less than 5, show the first 5 pages, with the last page separated by ellipsis buttons
		// if the current page is greater than the total number of pages minus 5, show the last 5 pages, with the first page separated by ellipsis buttons
		// otherwise, show the current page and the 2 pages before and after it, with the first and last page separated by ellipsis buttons

		const pagesToShow = 5;

		if (totalPages <= pagesToShow) {
			pages = Array.from({ length: totalPages }, (_, i) => ({
				name: (i + 1).toString(),
				href: `?page=${i + 1}`,
				active: i === currentPage,
			}));
		} else if (currentPage < pagesToShow) {
			pages = [
				...Array.from({ length: pagesToShow }, (_, i) => ({
					name: (i + 1).toString(),
					href: `?token=${token}&page=${i + 1}`,
					active: i === currentPage,
				})),
				{
					name: '…',
					href: `?token=${token}&page=${totalPages}`,
					active: false,
				},
				{
					name: totalPages.toString(),
					href: `?token=${token}&page=${totalPages}`,
					active: false,
				},
			];
		} else if (currentPage > totalPages - pagesToShow) {
			pages = [
				{
					name: '1',
					href: `?token=${token}&page=1`,
					active: false,
				},
				{
					name: '…',
					href: `?token=${token}&page=1`,
					active: false,
				},
				...Array.from({ length: pagesToShow }, (_, i) => ({
					name: (totalPages - pagesToShow + i + 1).toString(),
					href: `?token=${token}&page=${totalPages - pagesToShow + i + 1}`,
					active: totalPages - pagesToShow + i === currentPage,
				})),
			];
		} else {
			pages = [
				{
					name: '1',
					href: `?token=${token}&page=1`,
					active: false,
				},
				{
					name: '…',
					href: `?token=${token}&page=1`,
					active: false,
				},
				...Array.from({ length: 3 }, (_, i) => ({
					name: (currentPage - 1 + i + 1).toString(),
					href: `?token=${token}&page=${currentPage - 1 + i + 1}`,
					active: currentPage - 1 + i === currentPage,
				})),
				{
					name: '…',
					href: `?token=${token}&page=${totalPages}`,
					active: false,
				},
				{
					name: totalPages.toString(),
					href: `?token=${token}&page=${totalPages}`,
					active: false,
				},
			];
		}
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
		<SortableTableHeadCell
			{handleSort}
			{sortColumn}
			{sortDirection}
			column="length"
		>
			USERNAME
		</SortableTableHeadCell>
		<SortableTableHeadCell
			{handleSort}
			{sortColumn}
			{sortDirection}
			column="frequency"
		>
			FREQUENCY
		</SortableTableHeadCell>
		<SortableTableHeadCell
			{handleSort}
			{sortColumn}
			{sortDirection}
			column="verifiedAt"
		>
			CHECKED
		</SortableTableHeadCell>
		<SortableTableHeadCell
			{handleSort}
			{sortColumn}
			{sortDirection}
			column="updatedAt"
		>
			UPDATED
		</SortableTableHeadCell>
	</TableHead>
	<TableBody tableBodyClass="divide-y">
		{#each data as row}
			<TableBodyRow>
				<TableBodyCell>
					{row.username}
					{#if row.definition.length}
						<InformationCircle
							class="w-5 h-5 inline fill-blue-100 dark:fill-blue-200"
							color=""
							variation="solid"
						/>
						<Popover
							placement="right"
							title="Definition"
							class="w-64 whitespace-normal"
							arrow={false}
						>
							{row.definition[0]}
						</Popover>
					{/if}
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
