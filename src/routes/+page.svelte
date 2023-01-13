<script lang="ts">
	import Grid from 'gridjs-svelte';
	import dayjs from 'dayjs';
	import relativeTime from 'dayjs/plugin/relativeTime';
	import { onMount } from 'svelte';

	let token = '';

	onMount(() => {
		const params = new URL(window.location.href);
		token = params.searchParams.get('token')!;
	});

	dayjs.extend(relativeTime);

	const indexToColumn = ['length', 'frequency', 'verifiedAt', 'updatedAt'];
	const columns = [
		{
			name: 'Username',
			width: '10%',
			fixedHeader: true,
			resizable: true,
		},
		{
			name: 'Frequency',
			formatter: (cell: number) => {
				return cell.toFixed(2);
			},
			width: '10%',
			fixedHeader: true,
			resizable: true,
		},
		{
			name: 'Checked',
			formatter: (cell: string) => {
				return dayjs(cell).fromNow();
			},
			width: '10%',
		},
		{
			name: 'Updated',
			formatter: (cell: string) => {
				return dayjs(cell).fromNow();
			},
			width: '10%',
		},
		{
			name: 'Definition',
			formatter: (cell: string[]) => {
				return cell.join('\n');
			},
			sort: false,
			width: '50%',
			fixedHeader: true,
			resizable: true,
		},
	];
</script>

<div class="paginated">
	<Grid
		{columns}
		search={{
			debounceTimeout: 500,
			server: {
				url: (prev, keyword) => `${prev}&search=${keyword}`,
			},
		}}
		pagination={{
			enabled: true,
			limit: 10,
			server: {
				url: (prev, page, limit) =>
					`${prev}&offset=${page * limit}&limit=${limit}`,
			},
		}}
		server={{
			url: `https://api.matteopolak.com/names?token=${token}`,
			then: data =>
				// @ts-ignore
				data.results.map(profile => [
					profile.username,
					profile.frequency,
					profile.verifiedAt,
					profile.updatedAt,
					profile.definition,
				]),
			total: data => data.total,
		}}
		sort={{
			multiColumn: false,
			server: {
				url: (prev, columns) => {
					return `${prev}&sort=${
						columns[0]?.direction === 1 ? 'asc' : 'desc'
					}&column=${indexToColumn[columns[0]?.index ?? 1]}`;
				},
			},
		}}
	/>
</div>

<style global>
	@import 'https://cdn.jsdelivr.net/npm/gridjs/dist/theme/mermaid.min.css';
	@import url('https://fonts.googleapis.com/css2?family=Open+Sans:wght@500&display=swap');

	.paginated {
		font-family: 'Open Sans', sans-serif;
		font-size: 1rem;
	}
</style>
