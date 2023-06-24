<script lang="ts">
	import { onMount } from 'svelte';
	import { twMerge } from 'tailwind-merge';
	import './styles.css';

	import wordsData from '../data/words';

	let fetchingWords = false;
	let words: { value: string; hit: boolean | null; correct: boolean | null }[] = [];
	let wordIndex = 0;
	let highlightError = false;
	let input = '';
	let correctWords = 0;
	let incorrectWords = 0;
	let keystrokes = 0;
	let timer = 60;
	let interval: any;
	let game = {
		start: false,
		over: false
	};

	function getNewWords() {
		fetchingWords = true;
		setTimeout(() => {
			let list = [];
			for (let i = 0; i < 200; i++) {
				const random = Math.floor(Math.random() * wordsData.length - 1);
				list.push(wordsData[random]);
			}

			list = list.map((el) => ({ value: el, hit: null, correct: null }));

			words = list;
			fetchingWords = false;
		}, 500);
	}

	function startGame() {
		if (!game.start) game.start = true;
		game = { ...game };
		interval = setInterval(() => {
			if (timer <= 0) endGame();
			--timer;

			if (timer === 0 || game.over) {
				endGame();
			}
		}, 1000);
	}

	function endGame() {
		clearInterval(interval);
		game = { ...game, over: false };
		input = '';
		alert(JSON.stringify({ correctWords, incorrectWords, keystrokes }, null, 2));
	}

	function restartGame() {
		getNewWords();
		endGame();

		game = { start: false, over: false };
		timer = 60;
		wordIndex = 0;
	}

	function handleInput(event: any) {
		if (!game.start) {
			startGame();
		}
		keystrokes++;
		const element = event.target as HTMLInputElement;
		input = element.value;
		const keystroke = event.data;

		highlightError = input !== words[wordIndex].value.slice(0, input.length);

		if (keystroke === ' ') {
			if (input.trim() === words[wordIndex].value) {
				correctWords++;
				words[wordIndex].correct = true;
			} else {
				incorrectWords++;
				words[wordIndex].correct = false;
			}
			wordIndex++;
			highlightError = false;
			input = '';
		}
	}

	onMount(() => {
		getNewWords();
	});
</script>

<svelte:head>
	<title>Type App Svelte</title>
	<meta name="description" content="Svelte app for typing practice" />
</svelte:head>

<section class="h-screen w-screen flex justify-center items-center">
	<div class="flex flex-col items-center gap-4">
		<div class="flex gap-2">
			<p class="px-3 py-2 rounded border shadow text-center font-semibold">{timer}</p>
			<input
				placeholder={game.start ? '' : 'Start typing...'}
				class="w-full max-w-md shadow font-semibold border rounded px-3 py-2"
				type="text"
				value={input}
				on:input={handleInput}
			/>
		</div>
		{#if fetchingWords}
			loading...
		{:else}
			<div class="relative mt-4 font-semibold text-2xl max-w-3xl break-keep h-32 overflow-hidden">
				<div class="flex flex-wrap">
					{#each words as word, index}
						<div
							class={twMerge(
								'px-2 h-10 flex items-center rounded transition-transform',
								wordIndex === index && 'shadow-md -translate-y-1',
								wordIndex === index && (highlightError ? 'bg-red-500' : 'bg-gray-200'),
								words[index].correct
									? 'text-green-500'
									: words[index].correct != null && 'text-red-500'
							)}
						>
							{word.value}
						</div>
					{/each}
				</div>
				<span
					class="block absolute bottom-0 inset-x-0 h-16 bg-gradient-to-t from-white to-transparent"
				/>
			</div>
		{/if}
	</div>
</section>

<style style="display: block" visibility>
	.dale {
		background: #bada55;
	}
</style>
