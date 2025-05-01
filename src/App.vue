<script setup lang="ts">
import symbols from "./symbols";
import MemoryCard from "./components/MemoryCard.vue";
import { computed, ref, watch } from "vue";
import _ from "lodash";

const cards = ref([...symbols, ...symbols]);
const moves = ref(0);
const matches = ref(0);
const totalPairs = symbols.length * 2;
const openedCards = ref<Set<number>>(new Set());
const areTwoOpenedCard = computed(() => openedCards.value.size === 2);
const matchedCards = ref<Set<number>>(new Set<number>());
const isGameWon = computed(() => matches.value === totalPairs);

watch(areTwoOpenedCard, (value) => {
    if (!value) {
        return;
    }

    const [firstCardIndex, secondCardIndex] = openedCards.value;
    const firstCard = cards.value[firstCardIndex];
    const secondCard = cards.value[secondCardIndex];

    if (firstCard.name === secondCard.name) {
        setTimeout(() => {
            matchedCards.value.add(firstCardIndex);
            matchedCards.value.add(secondCardIndex);
            openedCards.value.clear();
            matches.value += 2;
        }, 200);
    } else {
        setTimeout(() => {
            openedCards.value.clear();
        }, 1000);
    }
});

const openCard = (cardId: number) => {
    openedCards.value.add(cardId);
    moves.value++;
};

const getCardStatus = (cardId: number) => {
    if (matchedCards.value.has(cardId)) {
        return "matched";
    }
    if (openedCards.value.has(cardId)) {
        return "opened";
    }
    return "closed";
};

const resetGame = () => {
    cards.value = _.shuffle([...symbols, ...symbols]);
    moves.value = 0;
    matches.value = 0;
    openedCards.value.clear();
    matchedCards.value.clear();
};

resetGame();
</script>

<template>
    <div id="app">
        <h1>Memory Game</h1>

        <div class="game-info">
            <div>Moves: {{ moves }}</div>
            <div>Matches: {{ matches }} / {{ totalPairs }}</div>
        </div>

        <div class="board">
            <MemoryCard
                v-for="({ name, image }, index) in cards"
                :key="index"
                :name
                :image
                :status="getCardStatus(index)"
                :disabled="areTwoOpenedCard"
                @click="openCard(index)"
            ></MemoryCard>
        </div>

        <button @click="resetGame">New Game</button>

        <div v-if="isGameWon" class="win-message">
            Congratulations! You won in {{ moves }} moves!
        </div>
    </div>
</template>

<style>
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: "Arial", sans-serif;
    background-color: #f4f7f9;
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
    padding: 20px;
}

#app {
    width: 100%;
    max-width: 800px;
    text-align: center;
}

h1 {
    color: #2c3e50;
    margin-bottom: 20px;
}

.game-info {
    margin-bottom: 20px;
    display: flex;
    justify-content: space-between;
    padding: 0 10px;
}

.game-info div {
    font-size: 1.2rem;
    font-weight: bold;
    color: #34495e;
}

.board {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    grid-gap: 15px;
    margin: 0 auto;
}

button {
    margin-top: 20px;
    padding: 10px 20px;
    background-color: #3498db;
    color: white;
    border: none;
    border-radius: 4px;
    font-size: 1rem;
    cursor: pointer;
    transition: background-color 0.3s;
}

button:hover {
    background-color: #2980b9;
}

.win-message {
    margin-top: 20px;
    font-size: 1.5rem;
    color: #27ae60;
    font-weight: bold;
}

@media (max-width: 600px) {
    .board {
        grid-template-columns: repeat(3, 1fr);
    }
}
</style>
