<template>
  <div id="app">
    <div v-if="user">
      <h1>Вітаємо, {{ user.first_name }}!</h1>
      <p>Ваш ID: {{ user.id }}</p>
      <div class="game-container">
        <div
          v-for="(tile, index) in tiles"
          :key="index"
          class="tile"
          :class="{ active: tile.active }"
          @click="tapTile(index)"
        ></div>
      </div>
      <div class="score">Рахунок: {{ score }}</div>
      <button @click="logout">Вийти</button>
    </div>
    <div v-else>
      <p>Будь ласка, увійдіть через Telegram.</p>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      user: null,
      tiles: Array(9).fill({ active: false }), // 9 клітинок
      score: 0,
      currentActive: null,
      interval: null,
    };
  },
  mounted() {
    this.initTelegramAuth();
    this.startGame();
  },
  beforeUnmount() {
    clearInterval(this.interval);
  },
  methods: {
    initTelegramAuth() {
      if (window.Telegram && Telegram.WebApp) {
        const user = Telegram.WebApp.initDataUnsafe.user;
        if (user) {
          this.user = user;
          this.saveUserData(user); // Зберегти дані користувача
        }
      }
    },
    async saveUserData(user) {
      try {
        const response = await fetch('https://your-backend-url.com/save-user', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify(user),
        });
        const result = await response.json();
        console.log('User data saved:', result);
      } catch (error) {
        console.error('Error saving user data:', error);
      }
    },
    startGame() {
      this.interval = setInterval(() => {
        this.resetTiles();
        const randomIndex = Math.floor(Math.random() * this.tiles.length);
        this.tiles[randomIndex].active = true;
        this.currentActive = randomIndex;
      }, 1000); // Змінює активну клітинку кожну секунду
    },
    resetTiles() {
      this.tiles = this.tiles.map(() => ({ active: false }));
    },
    tapTile(index) {
      if (index === this.currentActive) {
        this.score += 1;
        this.resetTiles();
      } else {
        this.score -= 1;
      }
    },
    logout() {
      this.user = null;
      localStorage.removeItem('tg_user');
      Telegram.WebApp.close(); // Закрити Web App
    },
  },
};
</script>

<style>
body {
  font-family: Arial, sans-serif;
  background-color: #f0f0f0;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  margin: 0;
}

#app {
  text-align: center;
}

.game-container {
  display: grid;
  grid-template-columns: repeat(3, 100px);
  grid-template-rows: repeat(3, 100px);
  gap: 10px;
  margin: 20px auto;
}

.tile {
  background-color: #ccc;
  border-radius: 10px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.tile.active {
  background-color: #ffcc00;
}

.score {
  margin-top: 20px;
  font-size: 24px;
}

button {
  margin-top: 20px;
  padding: 10px 20px;
  font-size: 16px;
  cursor: pointer;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 5px;
}

button:hover {
  background-color: #0056b3;
}
</style>