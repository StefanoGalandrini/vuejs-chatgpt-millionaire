<script>
import axios from "axios";
import AppResult from "./AppResult.vue";

export default {
	components: {
		AppResult,
	},

	data() {
		return {
			chatHistory: `Sei il conduttore del gioco "Chi vuol essere milionario". Devi fare una domanda partendo da un livello facile, e indicare quattro risposte diverse di cui una sola è la risposta corretta mentre le altre sono sbagliate. Ad esempio: "Qual è la capitale della Spagna?" - "A: Barcellona, B: Madrid, C: Malaga, D: Siviglia". Genera domande di massimo 25 parole su vari argomenti e risposte precedute dalle prime lettere dell'alfabeto (ad esempio: "A: Risposta 1"). Restituisci un oggetto con le seguenti chiavi in ordine: "question" con la domanda in formato stringa; "answers" con un array contenente le quattro risposte; "correct" con l'indice numerico della posizione nell'array della risposta corretta. Devono sempre esserci tutte e tre le chiavi richieste. Non devi aggiungere spiegazioni o altro testo. Ogni nuova domanda deve essere di maggiore difficoltà rispetto a quelle già fatte. Le domande già fatte saranno elencate di seguito per evitare ripetizioni e per dare un riferimento sulla difficoltà. `,

			currentImage: "",
			currentQuestion: "",
			currentAnswers: [],
			correct: null,
			level: 1,
			selectedAnswerIndex: null,
			baseUrl: "http://localhost:8000",
			chatEndpoint: "/api/chat",
			imageEndpoint: "/api/image",
			isLoading: false,
			isAnswerSelected: false,
		};
	},

	methods: {
		async startGame() {
			// Show Loader
			this.isLoading = true;

			let chatPayload = "";
			if (this.level === 1) {
				chatPayload = this.chatHistory;
			} else {
				chatPayload =
					this.chatHistory +
					" Prosegui generando un'altra domanda diversa da quelle già fatte e di maggiore difficoltà e restituisci l'oggetto con le chiavi richieste. ";
			}

			const chatResponseData = await this.makeServerRequest(
				this.chatEndpoint,
				chatPayload,
			);

			const chatResponse =
				typeof chatResponseData === "string"
					? JSON.parse(chatResponseData)
					: chatResponseData;

			if (chatResponse) {
				this.currentQuestion = chatResponse.question;
				this.currentAnswers = chatResponse.answers;
				this.correct = chatResponse.correct;
			} else {
				setTimeout(() => {
					this.startGame();
				}, 3000);
			}

			// Aggiunge la risposta alla chatHistory
			this.chatHistory += ` ${this.currentQuestion} `;

			const imagePayload = chatResponse.question;
			const imageResponse = await this.makeServerRequest(
				this.imageEndpoint,
				imagePayload,
			);
			this.currentImage = imageResponse;

			// Hide Loader
			this.isLoading = false;
			console.log(this.currentQuestion, this.correct + 1);
		},

		async makeServerRequest(endpoint, payload) {
			const fullUrl = this.baseUrl + endpoint;
			const response = await axios.post(fullUrl, {prompt: payload});
			return response.data.response;
		},

		getButtonClass(index) {
			return {
				selected: index === this.selectedAnswerIndex,
				discarded:
					this.selectedAnswerIndex !== null &&
					index !== this.selectedAnswerIndex,
			};
		},

		selectAnswer(index) {
			if (!this.isAnswerSelected) {
				this.selectedAnswerIndex = index;
				this.isAnswerSelected = true;
			}
		},

		prizeForLevel(l) {
			const prizes = [
				500, 1000, 2000, 5000, 10000, 25000, 50000, 75000, 100000, 250000,
				500000, 750000, 900000, 950000, 1000000,
			];
			return prizes[l - 1];
		},

		levelClass(l) {
			if (l === this.level) {
				return "current-level";
			} else if (l < this.level) {
				return "passed-level";
			}
			return "";
		},

		handleContinueGame(nextLevel) {
			this.level = nextLevel;
			this.currentImage = "";
			this.currentQuestion = "";
			this.currentAnswers = [];
			this.correct = null;
			this.isAnswerSelected = false;
			this.selectedAnswerIndex = null;
			this.startGame();
		},

		handleStopGame() {
			// Qui puoi gestire la logica per fermare il gioco
			// Per esempio, potresti mostrare la schermata finale, salvare il punteggio, ecc.
		},
	},

	mounted() {
		this.startGame();
	},
};
</script>

<template>
	<div class="game-grid">
		<!-- Overlay and Loader -->
		<div v-if="isLoading" class="overlay">
			<img src="../assets/images/loader.gif" alt="loader" />
		</div>

		<!-- Game Grid -->
		<div v-if="!isLoading">
			<div class="image-section">
				<img :src="currentImage" alt="" />
			</div>

			<div class="question-section">
				<p class="question">{{ currentQuestion }}</p>
				<div class="answers">
					<button
						class="answer"
						v-for="(answer, index) in currentAnswers"
						:key="index"
						:class="getButtonClass(index)"
						:disabled="isAnswerSelected"
						@click="selectAnswer(index)">
						{{ answer }}
					</button>
				</div>
			</div>
		</div>

		<!-- Prize Grid -->
		<div class="level-prize-section">
			<div v-for="l in 15" :key="l" :class="levelClass(l)">
				<span>Livello {{ l }}</span>
				<span>{{ prizeForLevel(l) }} €</span>
			</div>
		</div>

		<div v-if="isAnswerSelected">
			<AppResult
				:level="level"
				:selectedAnswerIndex="selectedAnswerIndex"
				:correctAnswerIndex="correct"
				@continueGame="handleContinueGame"
				@stopGame="handleStopGame" />
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

		img {
			width: 400px;
			margin-top: 2rem;
			border: 2px solid white;
			border-radius: 30px;
			overflow: hidden;
		}
	}

	.question-section {
		grid-row: 2 / 3;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;

		.question {
			margin-top: 3rem;
			font-size: 2rem;
		}

		.answers {
			margin-top: 2rem;
			width: 70%;
			display: flex;
			justify-content: center;
			flex-wrap: wrap;
			gap: 1.5rem;

			button {
				width: 40%;
				color: lightgrey;
				background-color: $color-button;
				padding: 0.5rem 1.5rem;
				font-size: 1.25rem;
				border: 1px solid grey;
				transition: all 250ms;

				&.selected {
					background-color: $color-chosen;
					color: white;
					box-shadow: 0 0 3px 3px rgba(255, 255, 255, 0.5);
					border: 1px solid white;
				}

				&.discarded {
					background-color: $color-discarded;
				}

				&:hover {
					color: white;
					box-shadow: 0 0 3px 3px rgba(255, 255, 255, 0.5);
					border: 1px solid white;
				}
			}
		}
	}

	.level-prize-section {
		grid-row: 1 / 3;
		grid-column: 2 / 3;
		padding: 3rem 4rem;
		display: flex;
		flex-direction: column-reverse;
		justify-content: space-around;

		div {
			display: flex;
			justify-content: space-between;
			color: lightgray;

			span {
				padding: 0.5rem 1.5rem;
			}

			&.current-level {
				font-weight: bold;
				color: white;
				background-color: $color-discarded;
				border-radius: 10px;
			}

			&.passed-level {
				color: $color-discarded;
				background-color: transparent;
			}
		}
	}
}

.overlay {
	position: fixed;
	inset: 0;
	background: rgba(0, 0, 0, 0.7);
	z-index: 99;
	display: flex;
	justify-content: center;
	align-items: center;

	img {
		width: 300px;
		z-index: 200;
	}
}
</style>
