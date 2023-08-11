<script>
import axios from "axios";

export default {
	data() {
		return {
			chatHistory: [],
			currentImage: "",
			currentQuestion: "",
			currentAnswers: [],
			level: 1,
			selectedAnswer: null,
		};
	},
	methods: {
		startGame() {
			axios
				.get("/api/getQuestion", {
					params: {
						chatHistory: this.chatHistory,
					},
				})
				.then((response) => {
					this.currentQuestion = response.data.question;
					this.currentAnswers = response.data.answers;
					this.currentImage = response.data.image;
					this.selectedAnswer = null; // Reset della risposta selezionata
				})
				.catch((error) => {
					console.error("Errore nel caricamento della domanda:", error);
				});
		},

		selectAnswer(index) {
			this.selectedAnswer = index;
		},

		prizeForLevel(l) {
			const prizes = [
				500, 1000, 2000, 5000, 10000, 25000, 50000, 75000, 100000, 250000,
				500000, 750000, 900000, 950000, 1000000,
			];
			return prizes[l - 1];
		},
	},

	mounted() {
		this.startGame();
	},
};
</script>

<template>
	<div class="game-grid">
		<!-- Game Grid -->
		<div class="image-section">
			<img :src="currentImage" alt="Domanda Immagine" />
		</div>
		<div class="question-section">
			<p>{{ currentQuestion }}</p>
			<div class="answers">
				<button
					v-for="(answer, index) in currentAnswers"
					:key="index"
					:class="{selected: selectedAnswer === index}"
					@click="selectAnswer(index)">
					{{ answer }}
				</button>
			</div>
		</div>

		<!-- Prize Grid -->
		<div class="level-prize-section">
			<div v-for="l in 15" :key="l" :class="{'current-level': l === level}">
				<span>Livello {{ l }}</span>
				<span>{{ prizeForLevel(l) }} €</span>
			</div>
		</div>
	</div>
</template>

<style lang="scss" scoped>
@import "../style.scss";
.game-grid {
	height: 100vh;
	display: grid;
	grid-template-columns: 3fr 1fr;
	grid-template-rows: 2fr 1fr;

	.image-section {
		grid-row: 1 / 2;
		display: flex;
		justify-content: center;
		align-items: center;
	}

	.question-section {
		grid-row: 2 / 3;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;

		.answers {
			display: flex;
			flex-direction: column;

			button {
				background-color: $color-bg;
				&:hover {
					background-color: $color-prize;
				}
				&.selected {
					background-color: green;
				}
			}
		}
	}

	.level-prize-section {
		grid-row: 1 / 3;
		grid-column: 2 / 3;
		padding: 3rem 6rem;
		display: flex;
		flex-direction: column-reverse;
		justify-content: space-around;
		align-items: space-between;

		div {
			display: flex;
			justify-content: space-between;

			&.current-level {
				font-weight: bold;
			}
		}
	}
}
</style>
