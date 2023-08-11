<script>
import axios from "axios";

export default {
	data() {
		return {
			chatHistory: [
				{
					role: "user",
					content:
						'Stiamo simulando il gioco "Chi vuole essere milionario?". In questo gioco, il concorrente deve rispondere a una serie di massimo 15 domande a difficoltà crescente,partendo da un livello molto facile, per vincere un premio massimo di un milione di euro. Per ogni domanda ci saranno quattro risposte, ma una sola è corretta. Ad esempio, se la domanda fosse "Qual è la capitale della Spagna", le risposte potrebbero essere "Barcellona, Madrid, Malaga, Siviglia". Tu sei il direttore del gioco, e senza mai fare riferimento a te stesso devi generare una domanda lunga al massimo 25 parole su un argomento a tua scelta, e fornire quattro possibili risposte indicando quale è la risposta corretta. Devi restituire come risposta solo un array con tre elementi: nel primo metterai solo la domanda, nel secondo un array con le quattro risposte, nel terzo l\'indice della risposta esatta. Non devi aggiungere altre spiegazioni. Ad ogni richiesta troverai tutti gli array che hai generato in modo da aumentare gradualmente la difficoltà delle domande. Inizia con la prima domanda, molto facile.',
				},
			],
			currentImage: "",
			currentQuestion: "",
			currentAnswers: [],
			level: 1,
			selectedAnswer: null,
			baseUrl: "http://localhost:8000",
			chatEndpoint: "/api/chat",
			imageEndpoint: "/api/image",
			isLoading: false,
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
			this.currentQuestion = chatResponse;
			console.log(chatResponse);

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
		<!-- Overlay and Loader -->
		<div v-if="isLoading" class="overlay">
			<img src="../assets/images/loader.gif" alt="loader" />
		</div>

		<!-- Game Grid -->
		<div v-if="!isLoading" class="image-section">
			<img :src="currentImage" alt="" />
		</div>
		<div v-if="!isLoading" class="question-section">
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
		// align-items: space-between;

		div {
			display: flex;
			justify-content: space-between;

			&.current-level {
				font-weight: bold;
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
