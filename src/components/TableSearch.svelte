<script lang="ts">
	import classNames from 'classnames';
	import {
		Checkbox,
		Chevron,
		ChevronLeft,
		ChevronRight,
		Dropdown,
		DropdownItem,
	} from 'flowbite-svelte';
	import FilterMenu from 'svelte-material-icons/FilterMenu.svelte';
	import Pagination from './Pagination.svelte';
	import type { LinkType } from 'flowbite-svelte/types';
	import { setContext } from 'svelte';
	import ShortTag from '../components/tags/ShortTag.svelte';
	import CommonTag from '../components/tags/CommonTag.svelte';
	import NewTag from '../components/tags/NewTag.svelte';
	import Heart from 'svelte-material-icons/Heart.svelte';
	import NameTag from '../components/tags/NameTag.svelte';
	import TakenTag from './tags/TakenTag.svelte';
	import BannedTag from './tags/BannedTag.svelte';

	export let divClass = 'relative overflow-x-auto shadow-md';
	export let inputValue = '';
	export let striped = false;
	export let hoverable = false;
	export let placeholder = 'Search';
	export let color: keyof typeof colors = 'default';
	export let totalPages = 0;
	export let currentPage = 0;
	export let pages: LinkType[] = [];
	export let pageSize = 10;
	export let tags: Set<string> = new Set();

	let dropdownOpen = false;
	const colors = {
		default: 'text-gray-500 dark:text-gray-400',
		blue: 'text-blue-100 dark:text-blue-100',
		green: 'text-green-100 dark:text-green-100',
		red: 'text-red-100 dark:text-red-100',
		yellow: 'text-yellow-100 dark:text-yellow-100',
		purple: 'text-purple-100 dark:text-purple-100',
		custom: '',
	};
	$: setContext('striped', striped);
	$: setContext('hoverable', hoverable);
	$: setContext('color', color);
</script>

<div class={divClass}>
	<div class="m-5">
		<label for="table-search" class="sr-only">Search</label>
		<div class="relative mt-1 flex flex-wrap gap-2">
			<div
				class="absolute top-3.5 left-0 flex items-center pt-0 pl-3 pointer-events-none"
			>
				<svg
					class="w-5 h-5 text-gray-500 dark:text-gray-400"
					fill="currentColor"
					viewBox="0 0 20 20"
					xmlns="http://www.w3.org/2000/svg"
					><path
						fill-rule="evenodd"
						d="M8 4a4 4 0 100 8 4 4 0 000-8zM2 8a6 6 0 1110.89 3.476l4.817 4.817a1 1 0 01-1.414 1.414l-4.816-4.816A6 6 0 012 8z"
						clip-rule="evenodd"
					/></svg
				>
			</div>
			<input
				bind:value={inputValue}
				type="text"
				id="table-search"
				class="p-3 bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-80 pl-10 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500"
				{placeholder}
			/>
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
					<ChevronLeft class="w-5 h-5" />
				</svelte:fragment>
				<svelte:fragment slot="next">
					<ChevronRight class="w-5 h-5" />
				</svelte:fragment>
			</Pagination>
			<ul>
				<li>
					<button
						class="text-center font-medium focus:ring-2 focus:outline-none inline-flex items-center justify-center px-3 py-3 text-sm font-medium border border-gray-300 text-gray-500 bg-white hover:bg-gray-100 hover:text-gray-700 dark:bg-gray-800 dark:border-gray-700 dark:text-gray-400 dark:hover:bg-gray-700 dark:hover:text-white rounded-lg"
						><Chevron>{pageSize}</Chevron></button
					>
					<Dropdown bind:open={dropdownOpen}>
						{#if pageSize !== 10}
							<DropdownItem
								on:click={() => (
									(pageSize = 10), (currentPage = 0), (dropdownOpen = false)
								)}
							>
								10
							</DropdownItem>
						{/if}
						{#if pageSize !== 15}
							<DropdownItem
								on:click={() => (
									(pageSize = 15), (currentPage = 0), (dropdownOpen = false)
								)}
							>
								15
							</DropdownItem>
						{/if}
						{#if pageSize !== 25}
							<DropdownItem
								on:click={() => (
									(pageSize = 25), (currentPage = 0), (dropdownOpen = false)
								)}
							>
								25
							</DropdownItem>
						{/if}
						{#if pageSize !== 50}
							<DropdownItem
								on:click={() => (
									(pageSize = 50), (currentPage = 0), (dropdownOpen = false)
								)}
							>
								50
							</DropdownItem>
						{/if}
						{#if pageSize !== 100}
							<DropdownItem
								on:click={() => (
									(pageSize = 100), (currentPage = 0), (dropdownOpen = false)
								)}
							>
								100
							</DropdownItem>
						{/if}
						{#if pageSize !== 250}
							<DropdownItem
								on:click={() => (
									(pageSize = 250), (currentPage = 0), (dropdownOpen = false)
								)}
							>
								250
							</DropdownItem>
						{/if}
					</Dropdown>
				</li>
			</ul>
			<ul>
				<li>
					<button
						class="text-center font-medium focus:ring-2 focus:outline-none inline-flex items-center justify-center px-3 py-3 text-sm font-medium border border-gray-300 text-gray-500 bg-white hover:bg-gray-100 hover:text-gray-700 dark:bg-gray-800 dark:border-gray-700 dark:text-gray-400 dark:hover:bg-gray-700 dark:hover:text-white rounded-lg"
						><FilterMenu height={20} /></button
					>
					<Dropdown class="w-44 p-3 space-y-3 text-sm">
						<li class="rounded p-2 hover:bg-gray-100 dark:hover:bg-gray-600">
							<Checkbox
								checked={tags.has('liked')}
								on:click={() => {
									if (tags.has('liked')) tags.delete('liked');
									else tags.add('liked');

									tags = tags;
									currentPage = 0;
								}}
							>
								<Heart color="#f56565" />
							</Checkbox>
						</li>
						<li class="rounded p-2 hover:bg-gray-100 dark:hover:bg-gray-600">
							<Checkbox
								checked={tags.has('name')}
								on:click={() => {
									if (tags.has('name')) tags.delete('name');
									else tags.add('name');

									tags = tags;
									currentPage = 0;
								}}
							>
								<NameTag />
							</Checkbox>
						</li>
						<li class="rounded p-2 hover:bg-gray-100 dark:hover:bg-gray-600">
							<Checkbox
								checked={tags.has('short')}
								on:click={() => {
									if (tags.has('short')) tags.delete('short');
									else tags.add('short');

									tags = tags;
									currentPage = 0;
								}}
							>
								<ShortTag />
							</Checkbox>
						</li>
						<li class="rounded p-2 hover:bg-gray-100 dark:hover:bg-gray-600">
							<Checkbox
								checked={tags.has('common')}
								on:click={() => {
									if (tags.has('common')) tags.delete('common');
									else tags.add('common');

									tags = tags;
									currentPage = 0;
								}}
							>
								<CommonTag />
							</Checkbox>
						</li>
						<li class="rounded p-2 hover:bg-gray-100 dark:hover:bg-gray-600">
							<Checkbox
								checked={tags.has('new')}
								on:click={() => {
									if (tags.has('new')) tags.delete('new');
									else tags.add('new');

									tags = tags;
									currentPage = 0;
								}}
							>
								<NewTag />
							</Checkbox>
						</li>
						<li class="rounded p-2 hover:bg-gray-100 dark:hover:bg-gray-600">
							<Checkbox
								checked={tags.has('taken')}
								on:click={() => {
									if (tags.has('taken')) tags.delete('taken');
									else tags.add('taken');

									tags = tags;
									currentPage = 0;
								}}
							>
								<TakenTag />
							</Checkbox>
						</li>
						<li class="rounded p-2 hover:bg-gray-100 dark:hover:bg-gray-600">
							<Checkbox
								checked={tags.has('banned')}
								on:click={() => {
									if (tags.has('banned')) tags.delete('banned');
									else tags.add('banned');

									tags = tags;
									currentPage = 0;
								}}
							>
								<BannedTag />
							</Checkbox>
						</li>
					</Dropdown>
				</li>
			</ul>
		</div>
	</div>
	<table
		{...$$restProps}
		class={classNames('w-full text-left text-sm', colors[color], $$props.class)}
	>
		<slot />
	</table>
</div>

<style>
	.sticky {
		position: sticky;
		top: 0rem;
	}
</style>
