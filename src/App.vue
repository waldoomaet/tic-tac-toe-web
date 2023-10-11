<template>
  <div class="container">
    <div class="row">
      <input type="text" v-model="host" placeholder="Host or IP" />
      <input type="text" v-model="port" placeholder="Port" />
      <button type="button" @click="connectToServer" :disabled="host.length < 0 || port.length < 0">Connect</button>
    </div>
    <div class="row">
      <div class="col"><button type="button" :disabled="disableButton(0, 0)" @click="() => onClick(0, 0)">{{ board[0][0]
      }}</button></div>
      <div class="col"><button type="button" :disabled="disableButton(0, 1)" @click="() => onClick(0, 1)">{{ board[0][1]
      }}</button></div>
      <div class="col"><button type="button" :disabled="disableButton(0, 2)" @click="() => onClick(0, 2)">{{ board[0][2]
      }}</button></div>
    </div>
    <div class="row">
      <div class="col"><button type="button" :disabled="disableButton(1, 0)" @click="() => onClick(1, 0)">{{ board[1][0]
      }}</button></div>
      <div class="col"><button type="button" :disabled="disableButton(1, 1)" @click="() => onClick(1, 1)">{{
        board[1][1] }}</button></div>
      <div class="col"><button type="button" :disabled="disableButton(1, 2)" @click="() => onClick(1, 2)">{{
        board[1][2] }}</button></div>
    </div>
    <div class="row">
      <div class="col"><button type="button" :disabled="disableButton(2, 0)" @click="() => onClick(2, 0)">{{ board[2][0]
      }}</button></div>
      <div class="col"><button type="button" :disabled="disableButton(2, 1)" @click="() => onClick(2, 1)">{{
        board[2][1] }}</button></div>
      <div class="col"><button type="button" :disabled="disableButton(2, 2)" @click="() => onClick(2, 2)">{{
        board[2][2] }}</button></div>
    </div>
    <div class="row">
      <p>{{ message }}</p>
      <button v-if="gameOver" type="button" @click="playAgain">Play again?</button>
    </div>
  </div>
</template>

<script>
import { Game, Status } from "./constants";
export default {
  data() {
    return {
      board: [
        [Game.none, Game.none, Game.none],
        [Game.none, Game.none, Game.none],
        [Game.none, Game.none, Game.none]
      ],
      host: "172.27.22.39",
      port: 13254,
      socket: null,
      player: null,
      myTurn: false,
      stopGame: true,
      gameOver: false,
      disableAll: true,
      message: ""
    }
  },
  methods: {
    disableButton(row, col) {
      return this.disableAll || this.board[row][col] !== Game.none;
    },
    boardFull() {
      for (let i = 0; i < this.board.length; i++) {
        for (let j = 0; j < this.board[i].length; j++) {
          if (this.board[i][j] == Game.none) {
            return false;
          }
        }
      }
      return true;
    },
    won(symbol) {
      for (let i = 0; i < this.board.length; i++) {
        if (this.board[i].every((x) => x == symbol)) {
          return true;
        }
      }

      // https://stackoverflow.com/questions/17428587/transposing-a-2d-array-in-javascript
      const transposedBoard = this.board[0].map((col, i) => this.board.map(row => row[i]));
      for (let i = 0; i < transposedBoard.length; i++) {
        if (transposedBoard[i].every((x) => x == symbol)) {
          return true;
        }
      }

      if ((this.board[0][0] == symbol && this.board[1][1] == symbol && this.board[2][2] == symbol) || (this.board[2][0] == symbol && this.board[1][1] == symbol && this.board[0][2] == symbol)) {
        return true;
      }

      return false;

    },
    playAgain() {
      this.socket.send(JSON.stringify({
        status: Status.playAgain
      }));
    },
    resetGame() {
      this.board = [
        [Game.none, Game.none, Game.none],
        [Game.none, Game.none, Game.none],
        [Game.none, Game.none, Game.none]
      ];
      if (this.player == Game.x) {
        this.myTurn = true;
        this.message = `Connected as player ${this.player}. You're first!`;
      } else {
        this.myTurn = false;
        this.message = `Connected as player ${this.player}. Player x is first!`;
      }
      this.stopGame = false;
      this.gameOver = false;
    },
    onClick(row, col) {
      if (this.myTurn && !this.stopGame && this.board[row][col] == Game.none) {
        this.board[row][col] = this.player;

        if (this.won(this.player)) {
          this.stopGame = true;
          this.socket.send(JSON.stringify({
            status: Status.gameOver,
            board: this.board
          }));
          this.message = `You won!`;
          this.gameOver = true;
          return;
        } else if (this.boardFull()) {
          this.stopGame = true;
          this.socket.send(JSON.stringify({
            status: Status.draw,
            board: this.board
          }));
          return;
        }

        this.myTurn = false;
        this.socket.send(JSON.stringify({
          status: Status.turnChange,
          board: this.board
        }));
        this.disableAll = true;
        this.message = "Waiting for the other player to play...";
      }
    },
    connectToServer() {
      this.socket = new WebSocket(`ws://${this.host}:${this.port}`);
      this.socket.onerror = () => {
        this.message = `Can't connect to ws://${this.host}:${this.port}`;
      }
      this.socket.onmessage = (event) => {
        const data = JSON.parse(event.data);
        switch (data.status) {
          case Status.connected:
            if (data.id == 0) {
              this.player = Game.x;
              this.myTurn = true;
            } else {
              this.player = Game.o;
              this.myTurn = false;
            }
            this.message = `Connected as player ${this.player}`;
            break;
          case Status.started:
            this.resetGame();
            if (this.player == Game.x) {
              this.message = `Connected as player ${this.player}. You're first!`;
            } else {
              this.message = `Connected as player ${this.player}. Player x is first!`;
            }
            this.stopGame = false;
            this.disableAll = false;
            break;
          case Status.turnChange:
            this.board = data.board;
            this.myTurn = true;
            this.message = "It's your turn!"
            this.disableAll = false;
            break;
          case Status.lose:
            this.stopGame = true;
            this.message = "Well... This is ackward... You lost...";
            this.gameOver = true;
            break;
          case Status.draw:
            this.stopGame = true;
            this.message = "It's a draw!";
            break;
          case Status.playAgain:
            this.resetGame();
            break;
        }
      };
    }
  },
  mounted() {
    console.log('Application mounted');
  },
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
