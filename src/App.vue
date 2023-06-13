<script setup>
import { computed, reactive, toRaw, watch } from 'vue';

const TIMEOUT_PLAYER_READY = 1000 * 10;

const player1 = reactive({
  id: 1,
  timerId: null,
  ready: false,
});
const player2 = reactive({
  id: 2,
  timerId: null,
  ready: false,
});
const isGameReady = computed(() => player1.ready && player2.ready);
const status = computed(() => {
  if (isGameReady.value) {
    return 'Start Game';
  }
  if (player1.ready && !player2.ready) {
    return 'Player 1 Ready';
  }
  if (!player1.ready && player2.ready) {
    return 'Player 2 Ready';
  }
  return 'Waiting';
});

watch(isGameReady, (ready) => {
  if (!ready) {
    return;
  }
  console.log('starting game', {
    player1: toRaw(player1),
    player2: toRaw(player2),
  });
  [player1, player2].forEach((p) => resetTimeout(p, true));
});

function resetTimeout(player, ready = false) {
  player.ready = ready;
  clearTimeout(player.timerId);
  player.timerId = null;
}

function onReadyPlayer(player) {
  console.log(`ready player ${player.id}`, toRaw(player));
  if (player.ready) {
    resetTimeout(player);
    return;
  }
  player.ready = true;
  player.timerId = setTimeout(() => resetTimeout(player), TIMEOUT_PLAYER_READY);
}

function onReset() {
  console.log('resetting game');
  [player1, player2].forEach((p) => resetTimeout(p, false));
}
</script>

<template>
  <div class="lobby">
    <header class="header">
      <h1>Cuttle: Async Rematch Logic</h1>
    </header>
    <section class="player player--one">
      <h2>Player 1</h2>
      <button
        :disabled="isGameReady"
        :class="{
          'button--ready': player1.ready,
        }"
        class="button"
        @click="() => onReadyPlayer(player1)"
      >
        Ready
      </button>
    </section>
    <section class="player player--two">
      <h2>Player 2</h2>
      <button
        :disabled="isGameReady"
        :class="{
          'button--ready': player2.ready,
        }"
        class="button"
        @click="() => onReadyPlayer(player2)"
      >
        Ready
      </button>
    </section>
    <footer class="footer">
      <h3>Status</h3>
      <p>{{ status }}</p>

      <h4>Reset</h4>
      <p>
        Once the game has started, you'll need to reset the state before you can
        try again.
      </p>
      <button @click="onReset">Reset</button>
    </footer>
  </div>
</template>

<style scoped>
.button {
  color: black;
  border: 2px solid black;
  padding: 0.5em 1em;
  margin: 0 auto;
  display: inline-block;
  background: white;
  cursor: pointer;
  font-weight: bold;
  text-transform: lowercase;
}

.button:disabled {
  cursor: not-allowed;
}

.button:not(:disabled):hover {
  border-style: dashed;
}

.button--ready {
  border-color: green;
  background: lightgreen;
}

.lobby {
  display: grid;
  grid-template-columns: 1fr 1fr;
  grid-template-rows: auto;
  grid-template-areas:
    'header header'
    'player-one player-two'
    'footer footer';
}

.header {
  grid-area: header;
}

.footer {
  grid-area: footer;
}

.player {
  background: lightgrey;
  margin: 0.5em;
  padding: 1em;
  text-align: center;
}

.player--one {
  grid-area: player-one;
}

.player--two {
  grid-area: player-two;
}
</style>
