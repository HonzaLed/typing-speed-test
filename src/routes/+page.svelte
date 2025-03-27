<script lang="ts">
	import { randomSentence } from '$lib/random-sentence';
	import { fade } from 'svelte/transition';
    import { browser } from '$app/environment';

	enum State {
		NotStarted,
		Started,
		Done
	}

	let sentence: string = $state(' ' + randomSentence());
    let wordCount: number = $derived(sentence.split(" ").length);
	let activeLetterIndex: number = $state(1);
	let activeLetter: string = $derived(sentence[activeLetterIndex]);
	let wrong: string = $state('');

	let typerState: State = $state(State.NotStarted);
	let startedInstant: Date | null = $state(null);
	let endedInstant: Date | null = $state(null);
    // let minutes: number = $state(0);
    // let seconds: number = $state(0);
	let duration: string = $state('');
    let wordsPerMinute: number = $state(0);

	function fillDurationString() {
		if (!endedInstant || !startedInstant) return;

		const durationUnix = endedInstant.getTime() - startedInstant.getTime();
		const totalSeconds = Math.floor(durationUnix / 1000);
		const minutes = Math.floor(totalSeconds / 60);
		const seconds = totalSeconds % 60;
		duration = `${String(minutes).padStart(2, '0')}:${String(seconds).padStart(2, '0')}`;
        console.log(`Words: ${wordCount}, seconds: ${totalSeconds}`);
        wordsPerMinute = Math.floor(wordCount / (totalSeconds / 60));
	}

	function keyDownHandler(event: KeyboardEvent & { currentTarget: EventTarget & HTMLDivElement }) {
		if (typerState === State.NotStarted) {
			typerState = State.Started;
			startedInstant = new Date();
		}
		if (activeLetterIndex === sentence.length - 1) {
			typerState = State.Done;
			endedInstant = new Date();
			fillDurationString();
		}

		if (event.key == activeLetter && wrong.length == 0) {
			activeLetterIndex++;
		} else {
			if (event.key == 'Backspace') {
				if (wrong.length !== 0) {
					// Delete one char from wrong
					wrong = wrong.substring(0, wrong.length - 1);
				} else {
					activeLetterIndex--;
				}
				return;
			}
			if (event.key.length !== 1) return;

			wrong += event.key;
			console.log(wrong);
		}
	}
</script>

<div class="relative flex h-svh w-screen flex-col items-center justify-center text-black dark:text-white bg-white dark:bg-black">
	{#if typerState === State.NotStarted || typerState === State.Started}
        <div
			class="flex p-8 lg:w-2/3 flex-col group"
			onkeydown={keyDownHandler}
			role="textbox"
			tabindex="0"
			out:fade={{duration: 0.3}}
			in:fade
		>
            <div class="flex w-full flex-row-reverse group-not-focus:blur-sm">
				{activeLetterIndex - 1}/{sentence.length - 1}
			</div>
			<div class="text-left text-2xl text-white/50 group-not-focus:blur-sm">
				{#each sentence as letter, index}
					<span class="relative" class:done={index < activeLetterIndex}
						>{letter}{#if index === activeLetterIndex - 1}
							{#if wrong.length !== 0}<span class="text-red-400">{wrong}</span>{/if}<div class="cursor absolute top-0 right-0 inline text-amber-300">|</div>
						{/if}</span
					>
				{/each}
			</div>
            <div class="absolute left-1/2 top-1/2 -translate-1/2 text-base group-focus:hidden">
                Click here to focus
            </div>
		</div>
	{/if}
	{#if typerState === State.Done}
		<div class="flex flex-col justify-center text-center items-center" out:fade in:fade={{delay: 0.3, duration: 0.3}}>
			<h1 class="text-4xl font-bold">Done!</h1>
			<p class="text-2xl font-medium">It took you: <span class="font-bold">{duration}</span></p>
            <p class="text-2xl font-medium">Thats {wordsPerMinute} WPM!</p>
            <button class="bg-black/50 dark:bg-white/50 rounded-full w-fit px-4 py-2 text-base mt-4 hover:cursor-pointer hover:bg-black/60 hover:dark:bg-white/60 transition-all" onclick={()=>{window.location.reload();}}>Repeat</button>
		</div>
	{/if}
</div>

<style lang="postcss">
	@reference "tailwindcss";

	.done {
		color: white;
	}
	.cursor {
		animation: 1s ease-in-out infinite cursorAnimation;
		width: 0.1rem;
		height: 1rem;
	}

	@keyframes cursorAnimation {
		0%,
		100% {
			opacity: 0;
		}
		50% {
			opacity: 1;
		}
	}
</style>
