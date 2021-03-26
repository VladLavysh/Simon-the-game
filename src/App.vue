<template>
  <div id="app" class="wrapper">
    <h1>Simon The Game</h1>
    <div class="game-board">
      <div class="simon">
        <div>
          <ul>
            <li
              v-for="(item, idx) of tilesOptions"
              :key="idx"
              class="tile"
              :class="[item.color, { active: tilesOptions[idx].active }]"
              @mousedown="tilesOptions[idx].active = true"
              @mouseup="tilesOptions[idx].active = false"
              @click="selectTile(idx)"
            ></li>
          </ul>
        </div>
      </div>
    </div>
    <div class="game-info">
      <h2>Раунд: {{ round }}</h2>
      <button @click="startGame">Start</button>
      <p v-if="gameOver">
        Вы проиграли, пройдя {{ complitedRounds }} раунда(ов)!
      </p>
    </div>
    <div class="game-options">
      <h2>Настройки сложности:</h2>
      <br />
      <label for="easy">Легкий</label>
      <input id="easy" type="radio" value="1.5" v-model="mode" checked /><br />
      <label for="normal">Средний</label>
      <input id="normal" type="radio" value="1" v-model="mode" /><br />
      <label for="hard">Тяжелый</label>
      <input id="hard" type="radio" value="0.4" v-model="mode" />
    </div>
  </div>
</template>

<script>
export default {
  name: "App",

  data: () => ({
    // mode: easy = 1.5, normal = 1.0, hard = 0.4
    mode: "1.5",
    round: 1,
    complitedRounds: 0,
    canChoose: false,
    gameOver: false,
    orderTileIdx: 0,
    tilesOrder: [],

    tilesOptions: [
      {
        color: "red",
        active: false,
        soundPath: "./assets/1.mp3",
      },
      {
        color: "blue",
        active: false,
        soundPath: "./assets/2.mp3",
      },
      {
        color: "yellow",
        active: false,
        soundPath: "./assets/3.mp3",
      },
      {
        color: "green",
        active: false,
        soundPath: "./assets/4.mp3",
      },
    ],
  }),

  methods: {
    startGame() {
      this.complitedRounds = 0;
      this.gameOver = false;
      this.createOrder();
    },

    createOrder() {
      setTimeout(() => {
        const rand = Math.round(1 - 0.5 + Math.random() * (4 - 1 + 1));
        this.tilesOrder.push(rand);

        // Включаем звук плитки
        const path = this.tilesOptions[rand - 1].soundPath;
        this.playSound(require(`${path}`));

        // Подсвечиваем плитки по случайной очередности
        this.tilesOptions[rand - 1].active = true;
        setTimeout(
          () => (this.tilesOptions[rand - 1].active = false),
          this.mode * 500
        );

        if (this.tilesOrder.length < this.round) {
          this.createOrder();
        } else {
          this.canChoose = true;
        }
      }, this.mode * 1000);
    },

    selectTile(idx) {
      if (!this.canChoose) return;

      const path = this.tilesOptions[idx].soundPath;
      idx++;

      this.playSound(require(`${path}`));

      // В случае проигрыша
      if (this.tilesOrder[this.orderTileIdx] !== idx) {
        this.gameOver = true;
        this.canChoose = false;
        this.tilesOrder = [];
        this.orderTileIdx = 0;
        this.round = 1;

        return;
      }

      this.orderTileIdx++;

      // В случае победы
      if (this.orderTileIdx === this.tilesOrder.length) {
        this.round++;
        this.complitedRounds++;
        this.tilesOrder = [];
        this.canChoose = false;
        this.orderTileIdx = 0;

        this.createOrder();
      }
    },

    playSound(path) {
      const audio = new Audio();
      audio.autoplay = true;
      audio.src = path;
      audio.play();
    },
  },
};
</script>

<style lang="scss">
@import url("./assets/main.css");

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
