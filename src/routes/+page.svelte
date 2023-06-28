<script lang="ts">
	import { onMount } from 'svelte';

	import './styles.css';

	import wordsData from '../data/words';
	import WordList from '../components/WordList.svelte';
	import Button from '../components/Button.svelte';
	import Results from '../components/Results.svelte';

	const TIMER_SECONDS = 60;
	const NUMBER_OF_WORDS = 200;

	let fetchingWords = false;
	let words: { value: string; hit: boolean | null; correct: boolean | null }[] = [];
	let wordIndex = 0;
	let highlightError = false;
	let input = '';
	let totalWords = 0;
	let correctWords: string[] = [];
	let wrongWords: string[] = [];
	let keystrokes = 0;
	let correctKeystrokes = 0;
	let wrongKeystrokes = 0;
	let timer = TIMER_SECONDS;
	let interval: any;
	let game = {
		start: false,
		over: false
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
		highlightError = false;
	}

	function restartGame() {
		getNewWords();
		endGame();

		game = { start: false, over: false };
		timer = TIMER_SECONDS;
		wordIndex = 0;
		totalWords = 0;
		correctWords = [];
		wrongWords = [];
		keystrokes = 0;
		correctKeystrokes = 0;
		wrongKeystrokes = 0;
	}

	function handleInput(event: any) {
		if (!game.start) {
			startGame();
		}
		const element = event.target as HTMLInputElement;
		input = element.value;
		const keystroke = event.data;

		highlightError = input !== words[wordIndex].value.slice(0, input.length);

		if (keystroke !== null && keystroke !== ' ') {
			keystrokes++;
			highlightError ? wrongKeystrokes++ : correctKeystrokes++;
		}

		if (keystroke === ' ') {
			totalWords++;
			if (input.trim() === words[wordIndex].value) {
				correctWords.push(words[wordIndex].value);
				words[wordIndex].correct = true;
			} else {
				wrongWords.push(words[wordIndex].value);
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

<section class="h-screen w-screen flex justify-center items-center bg-gray-900 text-gray-300">
	<div class="flex flex-col items-center px-4 font-medium">
		{#if game.over}
			<p class="text-2xl">Your results</p>
			<Results
				results={{
					header: {
						title: 'Accuracy',
						value: `${(
							(((correctWords.length / totalWords) * 100 || 0) +
								((correctKeystrokes / keystrokes) * 100 || 0)) /
							2
						).toFixed(2)}%`
					},
					results: [
						{ title: 'Total Words', value: totalWords },
						{
							title: 'WPM',
							value: (correctWords.length / (TIMER_SECONDS / 60)).toFixed(0),
							bgClass: 'bg-gray-700/50'
						},
						{ title: 'Correct Words', value: correctWords.length },
						{ title: 'Wrong Words', value: wrongWords.length },
						{
							title: 'Words accuracy',
							value: `${((correctWords.length / totalWords) * 100 || 0).toFixed(2)}%`
						},
						{ title: 'Total keystrokes', value: keystrokes },
						{
							title: 'KPM',
							value: (correctKeystrokes / (TIMER_SECONDS / 60)).toFixed(0),
							bgClass: 'bg-gray-700/50'
						},
						{ title: 'Correct Keystrokes', value: correctKeystrokes },
						{ title: 'Wrong Keystrokes', value: wrongKeystrokes },
						{
							title: 'Keystrokes accuracy',
							value: `${((correctKeystrokes / keystrokes) * 100 || 0).toFixed(2)}%`
						}
					]
				}}
			/>
			<div class="flex gap-4 mt-4">
				<Button outline disabled>Share</Button>
				<Button onClick={restartGame}>Reset</Button>
			</div>
		{/if}
		{#if !game.over}
			<div class="flex gap-2">
				<input
					disabled={game.over}
					placeholder={game.start ? '' : 'Start typing...'}
					class="bg-gray-800 w-full max-w-md shadow font-semibold border border-transparent rounded-md px-3 py-2"
					type="text"
					value={input}
					on:input={handleInput}
				/>
				<Button className="pointer-events-none">{timer}</Button>
			</div>
		{/if}
		{#if !game.over}
			<div class="h-40 mt-4">
				{#if fetchingWords}
					loading...
				{:else}
					<WordList {words} {wordIndex} {highlightError} />
				{/if}
			</div>
		{/if}
	</div>
</section>
