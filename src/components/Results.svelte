<script lang="ts">
	import { twMerge } from 'tailwind-merge';

	type TResult = {
		title: string;
		value: number | string;
		bgClass?: string;
	};

	type TResults = {
		header: TResult;
		results: TResult[];
	};
	export let results: TResults = {
		header: {
			title: '',
			value: 0
		},
		results: []
	};
</script>

<div class="flex flex-col gap-4 p-4 bg-gray-800 rounded-md overflow-hidden mt-4">
	<div class="flex flex-col text-center">
		<p>{results.header?.title}</p>
		<p class="text-4xl lg:text-6xl font-bold">{results.header?.value}</p>
	</div>
	<div class="grid grid-flow-col grid-rows-5 gap-4">
		{#each results.results as result}
			{#if result.title !== results.header?.title}
				<div class={twMerge('flex justify-between gap-4 px-3 py-2 rounded-md', result.bgClass)}>
					{result.title}:
					<span>{result.value}</span>
				</div>
			{/if}
		{/each}
	</div>
</div>
