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

  // TODO
  // There can be a poll here to check if the other player is ready every second, in which case
  // we would use a setInterval instead of a timeout and just query the game status
  // something like GET `/api/game/:id/rematch` which would send back the rematch status
  //     - Whether the players are ready, or if the game is no longer valid, etc
  //     - If the game is no longer valid, we can use that to trigger cleanup and either show
  //       a message to the user, or send them back to the lobby then show a message

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
      <p>
        This demo is to show how to manage the game start logic between two
        discrete clients. There are a few issues to keep in mind, though.
      </p>
      <ol>
        <li>Both players need to be ready before the game starts</li>
        <li>We need some way for client to be aware of the player readiness</li>
        <li>
          There needs to be some way to let the other player know that the game
          is starting
        </li>
      </ol>
      <p>
        To solve this, we need to use some sort of timeout in combination with
        either a check (or poll) to a data store to see if the other player is
        ready when we are, or to listen for a websocket event to notify when the
        game is ready.
      </p>
      <ol>
        <li>
          For a timeout, the logic can be done inside of the
          <code>onReadyPlayer</code> function
        </li>
        <li>
          If using websockets, there would need to be a socket listener for game
          ready via id but there's the potential the current user might miss the
          blast if they are disconnected.
        </li>
      </ol>
      <p>It's likely safer to just check the database on an interval.</p>
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
