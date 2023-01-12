<script lang="ts">
	import { onMount } from 'svelte';
	import InfiniteScroll from 'svelte-infinite-scroll';
	import { compute_rest_props } from 'svelte/internal';

	interface Profile {
		username: string;
		popularity: number;
	}

	let page = 0;
	let size = 100;
	let names: Profile[] = [];

	(async () => {
		const response = await fetch(
			`https://api.matteopolak.com/names?offset=${size * ++page}&limit=${size}`
		);

		names = await response.json();
	})();
</script>

<ul>
	{#each names as name}
		<li>{name.username} ({name.popularity.toFixed(2)})</li>
	{/each}
	<InfiniteScroll
		threshold={size}
		on:loadMore={async () => {
			console.log('load more');

			const response = await fetch(
				`https://api.matteopolak.com/names?offset=${
					size * ++page
				}&limit=${size}`
			);

			names.push(...(await response.json()));
			names = names;
		}}
	/>
</ul>

<style>
	ul {
		width: 400px;
		max-height: 400px;
		overflow-x: scroll;
	}
</style>
