<script lang="ts">
	import { onMount } from 'svelte';
	import {
		TableBody,
		TableBodyCell,
		TableBodyRow,
		TableHead,
		Popover,
		InformationCircle,
	} from 'flowbite-svelte';
	import SortableTableHeadCell from '../components/SortableTableHeadCell.svelte';
	import TableSearch from '../components/TableSearch.svelte';
	import HeartOutline from 'svelte-material-icons/HeartOutline.svelte';
	import Heart from 'svelte-material-icons/Heart.svelte';
	import Time from 'svelte-time';
	import { page } from '$app/stores';
	import type { LinkType } from 'flowbite-svelte/types';
	import ShortTag from '../components/tags/ShortTag.svelte';
	import CommonTag from '../components/tags/CommonTag.svelte';
	import NewTag from '../components/tags/NewTag.svelte';
	import NameTag from '../components/tags/NameTag.svelte';
	import TakenTag from '../components/tags/TakenTag.svelte';
	import BannedTag from '../components/tags/BannedTag.svelte';

	const enum Status {
		Unknown = 0,
		Available = 1,
		Taken = 2,
		Banned = 3,
		BatchAvailable = 4,
		BatchTaken = 4,
	}

	interface Row {
		username: string;
		frequency: number;
		verifiedAt: string;
		updatedAt: string;
		definition: string[] | null;
		liked: boolean;
		tags: string[] | null;
		status: Status;
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
	let tags: Set<string> = new Set();

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
			fetchData(
				currentPage,
				pageSize,
				searchTerm,
				sortColumn,
				sortDirection,
				tags
			);
		}
	}

	function likeName(username: string) {
		return fetch(`https://api.matteopolak.com/names/${username}/like`, {
			headers: {
				Authorization: token,
			},
		});
	}

	function dislikeName(username: string) {
		return fetch(`https://api.matteopolak.com/names/${username}/dislike`, {
			headers: {
				Authorization: token,
			},
		});
	}

	async function fetchData(
		currentPage: number,
		pageSize: number,
		searchTerm: string,
		sortColumn: string,
		sortDirection: string,
		tags: Set<string>
	) {
		const response = await fetch('https://api.matteopolak.com/names', {
			method: 'POST',
			body: JSON.stringify({
				offset: currentPage * pageSize,
				limit: pageSize,
				search: searchTerm || undefined,
				column: sortColumn,
				sort: sortDirection,
				tags: Array.from(tags),
			}),
			headers: {
				'Content-Type': 'application/json',
				Authorization: token,
			},
		});

		if (response.status !== 200) {
			data = [];
			totalPages = 1;

			return;
		}

		const json = await response.json();

		data = json.data;
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
	bind:tags
	{token}
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
					<button
						class="translate-y-0.5"
						on:click={() => {
							row.liked = !row.liked;
							row = row;

							if (row.liked) {
								likeName(row.username);
							} else {
								dislikeName(row.username);
							}
						}}
					>
						{#if row.liked}
							<Heart color="#f56565" />
						{:else}
							<HeartOutline color="#f56565" />
						{/if}
					</button>
					<span class="pl-2">
						{row.username}
					</span>
					{#if row.definition?.length}
						<InformationCircle
							class="w-5 h-5 inline fill-blue-300 dark:fill-blue-200"
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
						{#if row.tags?.includes('name')}
							<NameTag />
						{/if}
						{#if row.username.length <= 7}
							<ShortTag />
						{/if}
						{#if row.frequency >= 0.5}
							<CommonTag />
						{/if}
						{#if new Date(row.updatedAt).getTime() >= Date.now() - 86_400_000}
							<NewTag />
						{/if}
						{#if row.status === Status.Taken || row.status === Status.BatchTaken}
							<TakenTag />
						{/if}
						{#if row.status === Status.Banned}
							<BannedTag />
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
