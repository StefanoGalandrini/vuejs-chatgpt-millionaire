<script>
import AppGame from "./components/AppGame.vue";

export default {
	components: {
		AppGame,
	},
	data() {
		return {
			gameStarted: false,
		};
	},
	methods: {
		startGame() {
			this.gameStarted = true;
		},
	},
};
</script>

<template>
	<div v-if="!gameStarted" class="intro">
		<h1>Chi vuole essere milionario?</h1>
		<img src="./assets/images/logo.png" alt="" />
		<p>Sei pronto a mettere alla prova le tue conoscenze?</p>
		<button @click="startGame">Comincia il Gioco</button>
	</div>

	<AppGame v-if="gameStarted" />
</template>

<style lang="scss">
@import "./style.scss";

.intro {
	height: 100vh;
	display: flex;
	flex-direction: column;
	justify-content: center;
	align-items: center;

	h1 {
		text-align: center;
		font-weight: bold;
		text-transform: uppercase;
		margin-bottom: 3rem;
	}

	img {
		width: 300px;
		margin-bottom: 3rem;
	}

	button {
		margin: 2rem 0;
		border: none;
		background-color: $color-discarded;
		color: $color-bg;
		font-size: 1rem;
		padding: 0.8rem 1.5rem;
		border-radius: 50px;
		transition: all 250ms ease-in-out;

		&:hover {
			background-color: $color-prize;
			box-shadow: 0 0 2px 2px rgba(255, 255, 255, 0.7);
		}
	}
}
</style>
