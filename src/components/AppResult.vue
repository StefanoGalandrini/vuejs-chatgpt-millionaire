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
	<div>
		<p class="result-message">{{ getResultMessage() }}</p>
		<p v-if="correctAnswer">Livello: {{ nextLevel }}</p>
		<button v-if="correctAnswer && !gameOver" @click="continueGame">
			Prosegui
		</button>
		<button v-if="correctAnswer && !gameOver" @click="stopGame">Fermati</button>
	</div>
</template>

<style></style>
