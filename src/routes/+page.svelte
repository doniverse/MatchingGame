<script lang="ts">
	import { emoji } from './emoji';

	type State = 'start' | 'playing' | 'paused' | 'won' | 'lost';
	type Difficulty = 'easy' | 'medium' | 'hard';

	let state: State = 'start';
	let difficulty: Difficulty = 'easy';
	let size = 20;
	let grid = createGrid();
	let maxMatches = grid.length / 2;
	let selected: number[] = [];
	let matches: string[] = [];
	let timerId: number | null = null;
	let time = 60;

	function startGameTimer() {
		function countdown() {
			state !== 'paused' && (time -= 1);
		}
		timerId = setInterval(countdown, 1000);
	}

	function createGrid() {
		let cards = new Set<string>();
		let maxSize = size / 2;

		while (cards.size < maxSize) {
			const randomIndex = Math.floor(Math.random() * emoji.length);
			cards.add(emoji[randomIndex]);
		}
		return shuffle([...cards, ...cards]);
	}

	function shuffle<Items>(array: Items[]) {
		return array.sort(() => Math.random() - 0.5);
	}

	function selectCard(cardIndex: number) {
		selected = selected.concat(cardIndex);
	}

	function matchCards() {
		const [first, second] = selected;
		if (grid[first] === grid[second]) {
			matches = matches.concat(grid[first]);
		}

		setTimeout(() => (selected = []), 300);
	}

	function resetGame() {
		timerId && clearInterval(timerId);
		grid = createGrid();
		maxMatches = grid.length / 2;
		selected = [];
		matches = [];
		timerId = null;
		time = 60;
	}

	function gameWon() {
		state = 'won';
		resetGame();
	}
	function gameLost() {
		state = 'lost';
		resetGame();
	}

	function pauseGame(e: KeyboardEvent) {
		if (e.key === 'Escape') {
			switch (state) {
				case 'playing':
					state = 'paused';
					break;
				case 'paused':
					state = 'playing';
					break;
			}
		}
	}

	function handleChange(e) {
		difficulty = e.target.value;
	}

	function startGame() {
		switch (difficulty) {
			case 'easy':
				time = 100;
				break;
			case 'medium':
				time = 70;
				break;
			case 'hard':
				time = 60;
				break;
		}
		state = 'playing';
	}

	$: if (state === 'playing') {
		!timerId && startGameTimer();
	}

	$: selected.length === 2 && matchCards();
	$: maxMatches === matches.length && gameWon();
	$: time <= 0 && gameLost();
</script>

<svelte:window on:keydown={pauseGame} />

{#if state === 'paused'}
	<h1>Game Paused</h1>
{/if}
{#if state === 'start'}
	<h1>Matching Game</h1>
	<div class="difficultyRadios">
		<label for="easy">
			<input type="radio" name="difficulty" id="easy" value="easy" checked on:change={handleChange} />
			Easy
		</label>

		<label for="medium">
			<input type="radio" name="difficulty" id="medium" value="medium" on:change={handleChange} />
			Medium
		</label>

		<label for="hard">
			<input type="radio" name="difficulty" id="hard" value="hard" on:change={handleChange} />
			Hard
		</label>
	</div>
	<button on:click={startGame}>Play</button>
{/if}
{#if state === 'playing'}
	<h1 class="timer" class:pulse={time < 10}>{time}</h1>
	<div class="matches">
		{#each matches as card}
			<div>{card}</div>
		{/each}
	</div>
	<div class="cards">
		{#each grid as card, cardIndex}
			{@const isSelected = selected.includes(cardIndex)}
			{@const isSelectedOrMatched = selected.includes(cardIndex) || matches.includes(card)}
			{@const match = matches.includes(card)}
			<button
				class="card"
				class:selected={isSelected}
				class:flip={isSelectedOrMatched}
				disabled={isSelectedOrMatched}
				on:click={() => selectCard(cardIndex)}
			>
				<div class="back" class:match>
					{card}
				</div>
			</button>
		{/each}
	</div>
{/if}

{#if state === 'lost'}
	<h1>You lost! 💩</h1>
	<button on:click={() => (state = 'playing')}>Play again</button>
	<button on:click={() => (state = 'start')}>Menu</button>
{/if}

{#if state === 'won'}
	<h1>You Win! 🎉</h1>
	<button on:click={() => (state = 'playing')}>Play again</button>
	<button on:click={() => (state = 'start')}>Menu</button>
{/if}

<style>
	.cards {
		display: grid;
		grid-template-columns: repeat(5, 1fr);
		gap: 0.4rem;
	}

	.card {
		height: 140px;
		width: 140px;
		font-size: 4rem;
		background-color: var(--bg-2);
		transition: rotate 0.3s ease-out;
		transform-style: preserve-3d;

		&.flip {
			rotate: y 180deg;
			pointer-events: none;
		}

		& .back {
			position: absolute;
			inset: 0;
			display: grid;
			place-content: center;

			backface-visibility: hidden;
			rotate: y 180deg;
		}

		&.selected {
			border: 4px var(--border) solid;
		}

		& .match {
			transition: opacity 0.3s ease-out;
			opacity: 0.4;
		}
	}

	.matches {
		display: flex;
		gap: 1rem;
		margin-block: 2rem;
		font-size: 3rem;
	}
	.timer {
		transition: color 0.3s ease;
	}

	.pulse {
		color: var(--pulse);
		animation: pulse 1s infinite ease;
	}

	@keyframes pulse {
		to {
			scale: 1.4;
		}
	}

	.difficultyRadios {
		display: flex;
		flex-direction: column;
		padding: 0 4rem;
	}
	.difficultyRadios + button {
		margin-block: 2rem 0;
		border: 4px var(--border) dashed;
	}

	@media screen and (max-width: 820px){
		h1{
			font-size: 1.5rem;
		}
		.card {
			height: 80px;
			width: 80px;
			font-size: 2rem;
		}
		.matches {
			font-size: 1rem;
		}
	}

	@media screen and (max-width: 400px){
		h1{
			font-size: 1rem;
		}
		.card {
			height: 60px;
			width: 60px;
			font-size: 1.5rem;
		}
		.matches {
			font-size: 1rem;
		}
	}
</style>
