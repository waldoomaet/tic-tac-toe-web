<template>
  <div class="container">
    <div class="row">
      <div class="col"><button type="button" @click="() => onClick(0, 0)">{{ board[0][0] }}</button></div>
      <div class="col"><button type="button" @click="() => onClick(0, 1)">{{ board[0][1] }}</button></div>
      <div class="col"><button type="button" @click="() => onClick(0, 2)">{{ board[0][2] }}</button></div>
    </div>
    <div class="row">
      <div class="col"><button type="button" @click="() => onClick(1, 0)">{{ board[1][0] }}</button></div>
      <div class="col"><button type="button" @click="() => onClick(1, 1)">{{ board[1][1] }}</button></div>
      <div class="col"><button type="button" @click="() => onClick(1, 2)">{{ board[1][2] }}</button></div>
    </div>
    <div class="row">
      <div class="col"><button type="button" @click="() => onClick(2, 0)">{{ board[2][0] }}</button></div>
      <div class="col"><button type="button" @click="() => onClick(2, 1)">{{ board[2][1] }}</button></div>
      <div class="col"><button type="button" @click="() => onClick(2, 2)">{{ board[2][2] }}</button></div>
    </div>
    <div>
      <p>{{ message }}</p>
    </div>
  </div>
</template>

<script>
import { Game } from "./constants";
export default {
  data() {
    return {
      board: [
        [Game.none, Game.none, Game.none],
        [Game.none, Game.none, Game.none],
        [Game.none, Game.none, Game.none]
      ],
      player: Game.x,
      gameOver: false,
      message: "Game on!"
    }
  },
  methods: {
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
    onClick(row, col) {
      if (!this.gameOver && this.board[row][col] == Game.none) {
        this.board[row][col] = this.player;

        console.log(this.board);

        if (this.won(this.player)) {
          this.message = `Player ${this.player} won!`;
          this.gameOver = true;
        } else if (this.boardFull()) {
          this.message = "It's a draw!";
        }

        this.player = this.player == Game.x ? Game.o : Game.x;
      }
    },
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
