<template>
  <div class="game-container">
    <div class="p-5">
      <div class="row">
        <div class="col">
          <h5 class="text-light text-start p-0">
            Enter the IP address:
          </h5>
          <input
            class="field"
            type="text"
            v-model="host"
            placeholder="Host or IP"
          />
        </div>
        <div class="col">
          <h5 class="text-light text-start p-0">
            Enter the port number:
          </h5>
          <input class="field" type="text" v-model="port" placeholder="Port" />
        </div>
      </div>

      <div class="d-flex">
        <button
          class="connect-btn mt-2 p-2"
          type="button"
          @click="connectToServer"
          :disabled="host.length < 0 || port.length < 0"
        >
          <b>Connect</b>
        </button>
      </div>
      <div class="row mt-3">
        <b class="text-light h4">{{ message }}</b>
        <button v-if="gameOver" type="button" @click="playAgain">
          Play again?
        </button>
      </div>
      <div class="game-board p-5" v-if="twoPlayers">
        <div class="row board-row">
          <div class="col board-item border-bottom">
            <button
              class="board-btn"
              type="button"
              @click="() => onClick(0, 0)"
            >
              {{ board[0][0] }}
            </button>
          </div>
          <div class="col board-item border-left border-bottom">
            <button
              class="board-btn"
              type="button"
              @click="() => onClick(0, 1)"
            >
              {{ board[0][1] }}
            </button>
          </div>
          <div class="col board-item border-left border-bottom">
            <button
              class="board-btn"
              type="button"
              @click="() => onClick(0, 2)"
            >
              {{ board[0][2] }}
            </button>
          </div>
        </div>
        <div class="row board-row">
          <div class="col board-item border-bottom">
            <button
              class="board-btn"
              type="button"
              @click="() => onClick(1, 0)"
            >
              {{ board[1][0] }}
            </button>
          </div>
          <div class="col board-item border-left border-bottom">
            <button
              class="board-btn"
              type="button"
              @click="() => onClick(1, 1)"
            >
              {{ board[1][1] }}
            </button>
          </div>
          <div class="col board-item border-left border-bottom">
            <button
              class="board-btn"
              type="button"
              @click="() => onClick(1, 2)"
            >
              {{ board[1][2] }}
            </button>
          </div>
        </div>
        <div class="row board-row">
          <div class="col board-item">
            <button
              class="board-btn"
              type="button"
              @click="() => onClick(2, 0)"
            >
              {{ board[2][0] }}
            </button>
          </div>
          <div class="col board-item border-left">
            <button
              class="board-btn"
              type="button"
              @click="() => onClick(2, 1)"
            >
              {{ board[2][1] }}
            </button>
          </div>
          <div class="col board-item border-left">
            <button
              class="board-btn"
              type="button"
              @click="() => onClick(2, 2)"
            >
              {{ board[2][2] }}
            </button>
          </div>
        </div>
      </div>
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
        [Game.none, Game.none, Game.none],
      ],
      host: "172.27.22.39",
      port: 13254,
      socket: null,
      player: null,
      myTurn: false,
      stopGame: true,
      gameOver: false,
      disableAll: true,
      message: "",
      twoPlayers: false,
    };
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
      const transposedBoard = this.board[0].map((col, i) =>
        this.board.map((row) => row[i])
      );
      for (let i = 0; i < transposedBoard.length; i++) {
        if (transposedBoard[i].every((x) => x == symbol)) {
          return true;
        }
      }

      if (
        (this.board[0][0] == symbol &&
          this.board[1][1] == symbol &&
          this.board[2][2] == symbol) ||
        (this.board[2][0] == symbol &&
          this.board[1][1] == symbol &&
          this.board[0][2] == symbol)
      ) {
        return true;
      }

      return false;
    },
    playAgain() {
      this.socket.send(
        JSON.stringify({
          status: Status.playAgain,
        })
      );
    },
    resetGame() {
      this.board = [
        [Game.none, Game.none, Game.none],
        [Game.none, Game.none, Game.none],
        [Game.none, Game.none, Game.none],
      ];
      if (this.player == Game.x) {
        this.myTurn = true;
        this.message = `Connected as player ${this.player}. You're first!`;
      } else {
        this.myTurn = false;
        this.message = `Connected as player ${this.player}. Player x is first!`;
      }
      this.twoPlayers = true;
      this.stopGame = false;
      this.gameOver = false;
    },
    onClick(row, col) {
      if (this.myTurn && !this.stopGame && this.board[row][col] == Game.none) {
        this.board[row][col] = this.player;

        if (this.won(this.player)) {
          this.stopGame = true;
          this.socket.send(
            JSON.stringify({
              status: Status.gameOver,
              board: this.board,
            })
          );
          this.message = `You won!`;
          this.gameOver = true;
          return;
        } else if (this.boardFull()) {
          this.stopGame = true;
          this.socket.send(
            JSON.stringify({
              status: Status.draw,
              board: this.board,
            })
          );
          return;
        }

        this.myTurn = false;
        this.socket.send(
          JSON.stringify({
            status: Status.turnChange,
            board: this.board,
          })
        );
        this.disableAll = true;
        this.message = "Waiting for the other player to play...";
      }
    },
    connectToServer() {
      this.socket = new WebSocket(`ws://${this.host}:${this.port}`);
      this.socket.onerror = () => {
        this.message = `Can't connect to ws://${this.host}:${this.port}`;
      };
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
            this.message = "It's your turn!";
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
    },
  },
  mounted() {
    console.log("Application mounted");
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
.game-container {
  width: 100%;
  height: 100vh;
  background-image: linear-gradient(to top, #30cfd0 0%, #330867 100%);
  .connect-btn {
    border: none;
    border-radius: 10px;
    width: 12rem;
    box-shadow: rgba(0, 0, 0, 0.4) 0px 2px 4px,
      rgba(0, 0, 0, 0.3) 0px 7px 13px -3px,
      rgba(0, 0, 0, 0.2) 0px -3px 0px inset;
      &:hover{
        transform:translateY(-2px);
        transition: all .2s;
        background-image: linear-gradient(to right top, #6682ac, #7d8ebf, #9699d0, #b1a4df, #ceafed);
      color:#fff;      }
    /* background-image: linear-gradient(to right top, #051937, #0e296a, #3b349c, #7733c8, #bd12eb); */
  }
  .field {
    padding: 10px;
    border-radius: 10px;
    border: none;
    margin: 5px 0;
    background-color: rgba(238, 235, 235, 0.624);
    width: 100%;
  }
  .board-item {
    /* margin:10px; */
    padding: 0;
    font-size: 2rem;
    .board-btn {
      height: 8rem;
      background-color: transparent;
      border: none;
      color: #fff;
      width: 100%;
    }
  }
  .border-left {
    border-left: 2px solid #fff;
  }
  .border-bottom {
    border-bottom: 2px solid #fff !important;
  }
}
</style>
