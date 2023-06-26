<script lang="ts">
	import { twMerge } from 'tailwind-merge';

	export let type: 'submit' | 'button' | 'reset' | null | undefined = 'button';
	export let className: string = '';
	export let size: 'sm' | 'md' | 'lg' = 'md';
	export let outline = false;
	export let disabled = false;

	export let onClick: (event: MouseEvent) => void = () => {};
</script>

<button
	class={twMerge(
		'font-medium rounded-md transition-colors disabled:pointer-events-none disabled:opacity-50',
		size === 'md' && 'text-base px-4 py-2',
		!outline && 'border border-transparent bg-gray-800 hover:bg-gray-700 shadow',
		outline && 'hover:bg-gray-800',
		className
	)}
	{disabled}
	{type}
	on:click={onClick}
>
	<slot>Button</slot>
</button>
