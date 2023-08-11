<script>
import axios from "axios";

export default {
	data() {
		return {
			chatHistory: [
				{
					role: "user",
					content:
						'Stiamo simulando il gioco "Chi vuole essere milionario?". In questo gioco, il concorrente deve rispondere a una serie di massimo 15 domande a difficoltà crescente,partendo da un livello molto facile, per vincere un premio massimo di un milione di euro. Per ogni domanda ci saranno quattro risposte, ma una sola è corretta. Ad esempio, se la domanda fosse "Qual è la capitale della Spagna", le risposte potrebbero essere "Barcellona, Madrid, Malaga, Siviglia". Tu sei il direttore del gioco, e senza mai fare riferimento a te stesso devi generare una domanda lunga al massimo 25 parole su un argomento a tua scelta, e fornire quattro possibili risposte indicando quale è la risposta corretta. Devi restituire come risposta solo un oggetto con tre elementi: nel primo metterai solo la domanda con la chiave "question"; nel secondo, con la chiave "anwers", un array con le quattro risposte precedute dalle prime quattro lettere dell\'alfabeto e i due punti con uno spazio dopo (esempio: "A: Risposta 1"); nel terzo, con la chiave "correct", l\'indice della risposta esatta. Non devi aggiungere altre spiegazioni. Ad ogni richiesta troverai tutti gli array che hai generato in modo da aumentare gradualmente la difficoltà delle domande. Inizia con la prima domanda, molto facile.',
				},
			],
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

			const lastMessageContent =
				this.chatHistory[this.chatHistory.length - 1].content;

			const chatPayload = {prompt: lastMessageContent};
			const chatResponse = await this.makeServerRequest(
				this.chatEndpoint,
				chatPayload,
			);
			this.currentQuestion = chatResponse.question;
			this.currentAnswers = chatResponse.answers;
			this.correct = chatResponse.correct;

			// Aggiunge la risposta alla chatHistory
			this.chatHistory.push({
				role: "system",
				content: chatResponse,
			});

			const imagePayload = {prompt: chatResponse[0]};
			const imageResponse = await this.makeServerRequest(
				this.imageEndpoint,
				imagePayload,
			);
			this.currentImage = imageResponse;

			// Hide Loader
			this.isLoading = false;
		},

		async makeServerRequest(endpoint, payload) {
			const fullUrl = this.baseUrl + endpoint;
			const response = await axios.post(fullUrl, payload);
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

		getResult() {
			if (this.selectedAnswerIndex === this.correct) {
				this.level++;
				return "CORRETTA!";
			} else {
				return "sbagliata...";
			}
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
			<p class="answered">La risposta è...</p>
			<p class="result">{{ getResult() }}</p>
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
