<template>
  <div class="game-container">
    <div class="p-5">
      <div class="row">
        <div class="col">
          <h5 class="text-light text-start p-0">Enter the IP address:</h5>
          <input
            class="field"
            type="text"
            v-model="host"
            placeholder="Host or IP"
          />
        </div>
        <div class="col">
          <h5 class="text-light text-start p-0">Enter the port number:</h5>
          <input class="field" type="text" v-model="port" placeholder="Port" />
        </div>
      </div>

      <div class="d-flex">
        <button
          class="primary-btn mt-2 p-2"
          type="button"
          @click="connectToServer"
          :disabled="host.length < 0 || port.length < 0 || twoPlayers"
        >
          <b>Connect</b>
        </button>
      </div>
      <div class="row mt-3">
        <b class="text-light h4">
          <svg
            v-if="wonIcon"
            xmlns="http://www.w3.org/2000/svg"
            width="35"
            height="35"
            fill="#00cc7a"
            class="bi bi-emoji-wink-fill"
            viewBox="0 0 16 16"
          >
            <path
              d="M8 0a8 8 0 1 1 0 16A8 8 0 0 1 8 0zM7 6.5C7 5.672 6.552 5 6 5s-1 .672-1 1.5S5.448 8 6 8s1-.672 1-1.5zM4.285 9.567a.5.5 0 0 0-.183.683A4.498 4.498 0 0 0 8 12.5a4.5 4.5 0 0 0 3.898-2.25.5.5 0 1 0-.866-.5A3.498 3.498 0 0 1 8 11.5a3.498 3.498 0 0 1-3.032-1.75.5.5 0 0 0-.683-.183zm5.152-3.31a.5.5 0 0 0-.874.486c.33.595.958 1.007 1.687 1.007.73 0 1.356-.412 1.687-1.007a.5.5 0 0 0-.874-.486.934.934 0 0 1-.813.493.934.934 0 0 1-.813-.493z"
            />
          </svg>
          <svg
            v-else-if="lostIcon"
            xmlns="http://www.w3.org/2000/svg"
            width="40"
            height="40"
            fill="#ff1a1a"
            class="bi bi-emoji-frown"
            viewBox="0 0 20 20"
          >
            <path
              d="M8 16A8 8 0 1 0 8 0a8 8 0 0 0 0 16zM7 6.5C7 7.328 6.552 8 6 8s-1-.672-1-1.5S5.448 5 6 5s1 .672 1 1.5zm-2.715 5.933a.5.5 0 0 1-.183-.683A4.498 4.498 0 0 1 8 9.5a4.5 4.5 0 0 1 3.898 2.25.5.5 0 0 1-.866.5A3.498 3.498 0 0 0 8 10.5a3.498 3.498 0 0 0-3.032 1.75.5.5 0 0 1-.683.183zM10 8c-.552 0-1-.672-1-1.5S9.448 5 10 5s1 .672 1 1.5S10.552 8 10 8z"
            />
          </svg>
          {{ message }}</b
        >
        <div class="d-flex justify-content-center">
        <button v-if="gameOver || isDraw" class="primary-btn p-2" type="button" @click="playAgain">
          <b>Click to play again!</b>
          
        </button>
      </div>
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
      host: "31.208.229.75",
      port: 13254,
      socket: null,
      player: null,
      myTurn: false,
      stopGame: true,
      gameOver: false,
      disableAll: true,
      message: "",
      twoPlayers: false,
      wonIcon: false,
      lostIcon: false,
      isDraw:false
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
      this.wonIcon = false;
      this.lostIcon = false;
      this.stopGame = false;
      this.isDrawing = false;
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
          this.message = `You won! Yaaayyyy!`;
          this.wonIcon = true;
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
            this.player = data.player;
            this.myTurn = this.player == Game.x;
            this.message = `Connected as player ${this.player}`;
            break;
          case Status.started:
            this.resetGame();
            if (this.player == Game.x) {
              this.message = `Connected as player ${this.player}. You're first!`;
            } else {
              this.message = `Connected as player ${this.player}. Player x is first!`;
            }
            this.twoPlayers = true;
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
            this.message = "Well... This is awkward... You lost...";
            this.lostIcon = true;
            this.gameOver = true;
            break;
          case Status.draw:
            this.stopGame = true;
            this.isDraw = true;
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

  .primary-btn {
    border: none;
    border-radius: 10px;
    width: 12rem;
    box-shadow: rgba(0, 0, 0, 0.4) 0px 2px 4px,
      rgba(0, 0, 0, 0.3) 0px 7px 13px -3px,
      rgba(0, 0, 0, 0.2) 0px -3px 0px inset;
    &:hover:enabled {
      transform: translateY(-2px);
      transition: all 0.2s;
      background-image: linear-gradient(
        to right top,
        #6682ac,
        #7d8ebf,
        #9699d0,
        #b1a4df,
        #ceafed
      );
      color: #fff;
    }
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
    border-left: 4px solid #fff;
  }

  .border-bottom {
    border-bottom: 4px solid #fff !important;
  }
}
</style>
