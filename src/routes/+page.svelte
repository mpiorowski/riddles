<script lang="ts">
	import riddles from '$lib/riddles.json';

	let currentIndex = $state(0);
	let userAnswer = $state('');
	let errorMessage = $state('');

	const currentRiddle = $derived(riddles[currentIndex]);
	const isLastRiddle = $derived(currentRiddle?.isLocation === true);

	function handleNext() {
		if (isLastRiddle) {
			return;
		}

		// Normalize answers for comparison (trim, lowercase)
		const normalizedAnswer = userAnswer.trim().toLowerCase();
		const correctAnswer = currentRiddle.answer?.toLowerCase();

		if (normalizedAnswer === correctAnswer) {
			// Correct answer, move to next riddle
			currentIndex++;
			userAnswer = '';
			errorMessage = '';
		} else {
			// Wrong answer
			errorMessage = 'Incorrect answer, try again!';
		}
	}

	function handleKeydown(event: KeyboardEvent) {
		if (event.key === 'Enter') {
			handleNext();
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
				<div class="form-control mb-6">
					<input
						type="text"
						placeholder="Enter your answer..."
						class="input input-bordered w-full text-base sm:text-lg h-14 sm:h-16"
						bind:value={userAnswer}
						onkeydown={handleKeydown}
						autocomplete="off"
						autocorrect="off"
						autocapitalize="off"
						spellcheck="false"
					/>
					{#if errorMessage}
						<label class="label">
							<span class="label-text-alt text-error text-base">{errorMessage}</span>
						</label>
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
