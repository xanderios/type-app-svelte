<script lang="ts">
	import { afterUpdate } from 'svelte';
	import { fade } from 'svelte/transition';
	import { twMerge } from 'tailwind-merge';

	type TWord = { value: string; hit: boolean | null; correct: boolean | null };

	export let words: TWord[];
	export let wordIndex: number;
	export let highlightError: boolean;

	let wordElement: null | HTMLElement;
	let wordsOffset = 0;

	afterUpdate(() => {
		wordsOffset = wordElement?.offsetTop || 0;
	});
</script>

<div
	transition:fade
	class="relative font-semibold text-2xl max-w-3xl break-keep h-full overflow-hidden p-1"
>
	<div class="flex flex-wrap transition-transform" style="transform: translateY(-{wordsOffset}px);">
		{#each words as word, index}
			{#if wordIndex === index}
				<div
					bind:this={wordElement}
					class={twMerge(
						'px-2 h-10 flex items-center rounded-md shadow-md -translate-y-1 transition-transform',
						highlightError ? 'bg-red-500' : 'highlight bg-gray-700'
					)}
				>
					{word.value}
				</div>
			{:else}
				<div
					class={twMerge(
						'px-2 h-10 flex items-center rounded-md transition-transform',
						words[index].correct ? 'text-green-500' : words[index].correct != null && 'text-red-500'
					)}
				>
					{word.value}
				</div>
			{/if}
		{/each}
	</div>
	<span
		class="block absolute bottom-0 inset-x-0 h-32 bg-gradient-to-t from-gray-900 to-transparent"
	/>
</div>
