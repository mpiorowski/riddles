<script lang="ts">
	import riddles from '$lib/riddles.json';

	let currentIndex = $state(0);
	let errorMessage = $state('');
	let inputs: HTMLInputElement[] = $state([]);

	const currentRiddle = $derived(riddles[currentIndex]);
	const isLastRiddle = $derived(currentRiddle?.isLocation === true);
	const answerLength = $derived(currentRiddle?.answer?.length || 0);

	let characters = $state<string[]>([]);

	$effect(() => {
		// Reset characters when riddle changes
		characters = Array(currentRiddle?.answer?.length || 0).fill('');
		inputs = [];
		errorMessage = '';

		// Focus first input on next tick
		setTimeout(() => {
			const firstInput = document.querySelector('input[type="text"]') as HTMLInputElement;
			firstInput?.focus();
		}, 0);
	});

	function handleInput(index: number, event: Event) {
		const input = event.target as HTMLInputElement;
		let value = input.value.toLowerCase();

		// Take only the last character if multiple were pasted or typed
		if (value.length > 0) {
			const newChar = value.slice(-1);
			characters[index] = newChar;
			input.value = newChar;

			// Move to next input automatically
			if (index < answerLength - 1) {
				// Find next input by querying the DOM
				const nextInput = input.parentElement?.children[index + 1] as HTMLInputElement;
				nextInput?.focus();
			}
		}
	}

	function handleKeydown(index: number, event: KeyboardEvent) {
		if (event.key === 'Backspace') {
			event.preventDefault();
			if (characters[index] !== '') {
				// Clear current input
				characters[index] = '';
			} else if (index > 0) {
				// Move to previous input and clear it
				const prevInput = (event.target as HTMLInputElement).parentElement?.children[index - 1] as HTMLInputElement;
				characters[index - 1] = '';
				prevInput?.focus();
			}
		} else if (event.key === 'ArrowLeft' && index > 0) {
			const prevInput = (event.target as HTMLInputElement).parentElement?.children[index - 1] as HTMLInputElement;
			prevInput?.focus();
		} else if (event.key === 'ArrowRight' && index < answerLength - 1) {
			const nextInput = (event.target as HTMLInputElement).parentElement?.children[index + 1] as HTMLInputElement;
			nextInput?.focus();
		} else if (event.key === 'Enter') {
			handleNext();
		}
	}

	function handlePaste(event: ClipboardEvent) {
		event.preventDefault();
		const pastedText = event.clipboardData?.getData('text').toLowerCase() || '';

		for (let i = 0; i < Math.min(pastedText.length, answerLength); i++) {
			characters[i] = pastedText[i];
		}

		// Focus the last filled input or the next empty one
		const nextIndex = Math.min(pastedText.length, answerLength - 1);
		inputs[nextIndex]?.focus();
	}

	function handleNext() {
		if (isLastRiddle) {
			return;
		}

		const userAnswer = characters.join('').toLowerCase();
		const correctAnswer = currentRiddle.answer?.toLowerCase();

		if (userAnswer === correctAnswer) {
			// Correct answer, move to next riddle
			currentIndex++;
			errorMessage = '';
		} else {
			// Wrong answer
			errorMessage = 'Incorrect answer, try again!';
			// Shake animation could be added here
		}
	}
</script>

<div class="min-h-screen flex items-center justify-center bg-base-200 p-4 sm:p-6">
	<div class="card w-full max-w-2xl bg-base-100 shadow-xl">
		<div class="card-body p-6 sm:p-8">
			<h2 class="card-title text-xl sm:text-2xl mb-6">
				Riddle {currentIndex + 1} of {riddles.length}
			</h2>

			<div class="bg-base-200 p-6 sm:p-8 rounded-lg mb-6">
				<p class="text-lg sm:text-xl leading-relaxed">{currentRiddle.question}</p>
			</div>

			{#if !isLastRiddle}
				<div class="mb-6">
					<div class="flex justify-center gap-2 sm:gap-3 flex-wrap">
						{#each Array(answerLength) as _, index}
							<input
								type="text"
								class="input input-bordered w-12 h-12 sm:w-14 sm:h-14 text-center text-xl sm:text-2xl font-bold uppercase"
								bind:this={inputs[index]}
								value={characters[index]}
								oninput={(e) => handleInput(index, e)}
								onkeydown={(e) => handleKeydown(index, e)}
								onpaste={handlePaste}
								autocomplete="off"
								autocorrect="off"
								autocapitalize="off"
								spellcheck="false"
							/>
						{/each}
					</div>
					{#if errorMessage}
						<div class="text-center mt-4">
							<span class="text-error text-base font-semibold">{errorMessage}</span>
						</div>
					{/if}
				</div>

				<div class="card-actions">
					<button class="btn btn-primary w-full text-base sm:text-lg h-14 sm:h-16" onclick={handleNext}>
						Next
					</button>
				</div>
			{:else}
				<div class="alert alert-success text-base sm:text-lg p-6">
					<svg
						xmlns="http://www.w3.org/2000/svg"
						class="stroke-current shrink-0 h-8 w-8 sm:h-10 sm:w-10"
						fill="none"
						viewBox="0 0 24 24"
					>
						<path
							stroke-linecap="round"
							stroke-linejoin="round"
							stroke-width="2"
							d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z"
						/>
					</svg>
					<span>Congratulations! You've completed all the riddles!</span>
				</div>
			{/if}
		</div>
	</div>
</div>
