<script lang="ts">
	import { onMount } from 'svelte';

	import './styles.css';

	import wordsData from '../data/words';
	import WordList from '../components/WordList.svelte';

	const TIMER_SECONDS = 5;
	const NUMBER_OF_WORDS = 200;

	let fetchingWords = false;
	let words: { value: string; hit: boolean | null; correct: boolean | null }[] = [];
	let wordIndex = 0;
	let highlightError = false;
	let input = '';
	let totalWords = 0;
	let correctWords = 0;
	let incorrectWords = 0;
	let keystrokes = 0;
	let timer = TIMER_SECONDS;
	let interval: any;
	let game = {
		start: false,
		over: true
	};

	function getNewWords() {
		fetchingWords = true;
		setTimeout(() => {
			let list = [];
			for (let i = 0; i < NUMBER_OF_WORDS; i++) {
				const random = Math.floor(Math.random() * wordsData.length - 1);
				list.push(wordsData[random]);
			}

			list = list.map((el) => ({ value: el, hit: null, correct: null }));

			words = list;
			fetchingWords = false;
		}, 500);
	}

	function startGame() {
		if (!game.start) game = { start: true, over: false };
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
		game = { start: false, over: true };
		input = '';
	}

	function restartGame() {
		getNewWords();
		endGame();

		game = { start: false, over: false };
		timer = TIMER_SECONDS;
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
			totalWords++;
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
	<div class="flex flex-col items-center gap-4 px-4">
		{#if game.over}
			<p class="font-semibold text-2xl">Your results</p>
			<div class="flex">
				<div class="flex flex-col text-center">
					<p class="text-4xl font-bold">{totalWords}</p>
					<p class="font-semibold">Total Words</p>
					<div class="text-left">
						<p class="text-green-500">
							<abbr class="no-underline" title="Words per minute">WPM</abbr>: {correctWords / 60}
						</p>
						<p>Correct words: {correctWords}</p>
						<p>Incorrect words: {incorrectWords}</p>
					</div>
				</div>

				<div class="flex flex-col text-center">
					<p class="text-4xl font-bold">{keystrokes}</p>
					<p class="font-semibold">Total Keystrokes</p>
					<div class="text-left">
						<p class="text-green-500">{correctWords}</p>
						<p>{incorrectWords}</p>
					</div>
				</div>
			</div>
		{/if}
		<div class="flex gap-2">
			{#if game.over}
				<button>Share</button>
				<button>Reset</button>
			{:else}
				<input
					disabled={game.over}
					placeholder={game.start ? '' : 'Start typing...'}
					class="w-full max-w-md shadow font-semibold border rounded px-3 py-2"
					type="text"
					value={input}
					on:input={handleInput}
				/>
				<p class="px-3 py-2 rounded border shadow text-center font-semibold">{timer}</p>
			{/if}
		</div>
		<div class="h-32">
			{#if fetchingWords}
				loading...
			{:else}
				<WordList {words} {wordIndex} {highlightError} />
			{/if}
		</div>
	</div>
</section>
