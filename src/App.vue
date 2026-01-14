<template>
  <div class="game-container">
    <div class="start-screen" v-if="!gameStarted">
      <el-button 
        type="primary" 
        size="large" 
        class="start-button"
        @click="startGame"
      >
        开始游戏
      </el-button>
    </div>
    <div v-else class="game-playing">
      <div class="status-bar">
        <div class="status-item">
          <span class="status-label">所在楼层</span>
          <span class="status-value">{{ player.floor }}</span>
        </div>
        <div class="status-divider"></div>
        <div class="status-item">
          <span class="status-label">HP</span>
          <span class="status-value">{{ player.hp }} / {{ player.maxHp }}</span>
        </div>
        <div class="status-item">
          <span class="status-label">状态</span>
          <span class="status-value">{{ player.status }}</span>
        </div>
        <div class="status-item">
          <span class="status-label">武器</span>
          <span class="status-value">{{ player.weapon || '空' }}</span>
        </div>
        <div class="status-item">
          <span class="status-label">护甲</span>
          <span class="status-value">{{ player.armor || '空' }}</span>
        </div>
      </div>
      <div class="vertical-line"></div>
      <div class="game-area">
        <p class="game-text">游戏开始中</p>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'

const gameStarted = ref(false)

const player = reactive({
  floor: 1,
  hp: 100,
  maxHp: 100,
  status: '正常',
  weapon: '',
  armor: ''
})

const startGame = () => {
  gameStarted.value = true
}
</script>

<style scoped>
.game-container {
  width: 100vw;
  height: 100vh;
  background: white;
}

.start-screen {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.game-playing {
  display: flex;
  width: 100%;
  height: 100%;
  align-items: stretch;
  position: relative;
}

.status-bar {
  display: flex;
  flex-direction: column;
  padding: 20px;
  min-width: 200px;
  text-align: left;
  position: absolute;
  top: 20%;
  left: 0;
}

.vertical-line {
  position: absolute;
  left: 240px;
  height: 100vh;
  width: 1px;
  background: #000;
}

.status-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 0;
}

.status-label {
  font-weight: bold;
  color: #000;
  font-size: 0.95rem;
}

.status-value {
  color: #000;
  font-size: 0.95rem;
}

.status-divider {
  height: 1px;
  background: #000;
  margin: 10px 0;
}

.game-area {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: scale(0.9);
  }
  to {
    opacity: 1;
    transform: scale(1);
  }
}

.start-button {
  font-size: 1.5rem;
  padding: 20px 60px;
  height: auto;
  border-radius: 12px;
  font-weight: bold;
  letter-spacing: 2px;
}

.game-text {
  font-size: 2rem;
  color: #000;
  font-weight: bold;
  margin: 0;
  animation: fadeIn 0.5s ease-in-out;
}
</style>
