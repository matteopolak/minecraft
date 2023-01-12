<script lang="ts">
	import Grid from 'gridjs-svelte';

	const columns = [
		{
			name: 'Username',
			sort: false,
		},
		{
			name: 'Popularity',
			formatter: (cell: number) => {
				return cell.toFixed(2);
			},
			sort: true,
		},
		{
			name: 'Definition',
		},
	];

	interface Profile {
		username: string;
		popularity: number;
	}
</script>

<Grid
	{columns}
	sort
	search={{
		debounceTimeout: 500,
		server: {
			url: (prev, keyword) => `${prev}search=${keyword}&`,
		},
	}}
	pagination={{
		enabled: true,
		limit: 15,
		server: {
			url: (prev, page, limit) =>
				`${prev}offset=${page * limit}&limit=${limit}&`,
		},
	}}
	server={{
		url: 'https://api.matteopolak.com/names?',
		then: data =>
			// @ts-ignore
			data.results.map(profile => [
				profile.username,
				profile.popularity,
				profile.definition,
			]),
		total: data => data.total,
	}}
/>

<style global>
	@import 'https://cdn.jsdelivr.net/npm/gridjs/dist/theme/mermaid.min.css';
</style>
