<script>
export default {
	props: {
		level: {
			type: Number,
			required: true,
		},
		selectedAnswerIndex: {
			type: Number,
			required: true,
		},
		correctAnswerIndex: {
			type: Number,
			required: true,
		},
	},

	computed: {
		correctAnswer() {
			return this.selectedAnswerIndex === this.correctAnswerIndex;
		},

		nextLevel() {
			return this.level + 1;
		},

		gameOver() {
			return this.level === 15;
		},
	},

	methods: {
		getResultMessage() {
			return this.correctAnswer ? "Risposta Corretta!" : "Risposta Sbagliata!";
		},

		continueGame() {
			this.$emit("continueGame", this.nextLevel);
		},

		stopGame() {
			this.$emit("stopGame");
		},
	},
};
</script>

<template>
	<div class="results" v-if="correctAnswer && !gameOver">
		<div class="result-message">
			<p class="result">{{ getResultMessage() }}</p>
			<p v-if="correctAnswer">Hai superato il livello {{ nextLevel - 1 }}</p>
		</div>
		<div class="buttons">
			<button @click="continueGame">Prosegui</button>
			<button @click="stopGame">Fermati</button>
		</div>
	</div>
</template>

<style lang="scss" scoped>
@import "../style.scss";
.results {
	width: 80%;
	margin-inline: auto;
	display: flex;
	flex-direction: column;
	justify-content: center;
	align-items: center;

	.result-message {
		font-size: 1.3rem;
		text-align: center;
		margin: 1rem auto;
	}

	.result {
		font-size: 1.75rem;
		text-transform: uppercase;
		font-weight: bold;
		margin: 0.5rem auto;
		color: $color-prize;
	}

	button {
		margin: 0.5rem 2rem;
		border: none;
		background-color: $color-prize;
		color: $color-bg;
		font-size: 1rem;
		padding: 0.8rem 1.5rem;
		border-radius: 50px;
		transition: all 250ms ease-in-out;

		&:hover {
			background-color: white;
			box-shadow: 0 0 2px 2px rgba(255, 255, 255, 0.7);
		}
	}
}
</style>
