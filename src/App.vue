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
    <div v-else class="game-playing" @keydown="handleKeyDown" tabindex="0">
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
        <div v-if="gameWon" class="game-won">游戏胜利</div>
        <div v-if="gameWon" class="reset-button-container">
          <el-button type="primary" size="small" @click="resetGame">重置游戏</el-button>
        </div>
        <div v-if="itemMessage" class="item-message">{{ itemMessage }}</div>
        <div class="game-map">
          <div 
            v-for="(row, rowIndex) in map" 
            :key="rowIndex" 
            class="map-row"
          >
            <div 
              v-for="(cell, colIndex) in row" 
              :key="colIndex" 
              class="map-cell"
              :class="{
                'wall': cell === 1,
                'player': cell === 2,
                'destination': cell === 3,
                'treasure': cell === 4
              }"
            >
              {{ getCellIcon(cell) }}
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, computed, onMounted, onUnmounted, nextTick } from 'vue'

const gameStarted = ref(false)
const gameWon = ref(false)
const map = ref([])
const gamePlayingRef = ref(null)
const itemMessage = ref('')

const player = reactive({
  floor: 1,
  hp: 100,
  maxHp: 100,
  status: '正常',
  weapon: '',
  armor: '',
  x: 1,
  y: 1
})

const loadMap = async () => {
  try {
    const response = await fetch('/maps/map1.txt')
    const text = await response.text()
    const lines = text.trim().split('\n')
    
    const mapData = []
    for (const line of lines) {
      const row = line.trim().split('').map(Number)
      mapData.push(row)
    }
    
    // 找到玩家位置（值为2）
    let playerPos = { row: 1, col: 1 }
    for (let row = 0; row < mapData.length; row++) {
      for (let col = 0; col < mapData[row].length; col++) {
        if (mapData[row][col] === 2) {
          playerPos = { row, col }
          break
        }
      }
      if (mapData[row][playerPos.col] === 2) break
    }
    
    return { mapData, playerPos }
  } catch (error) {
    console.error('加载地图失败：', error)
    // 失用数据
    return {
      mapData: [
        [1, 1, 1, 1, 1, 1, 1, 1, 1],
        [1, 0, 0, 0, 0, 1, 3, 0, 0, 0, 1],
        [1, 0, 0, 0, 1, 0, 1, 0, 0, 0, 1],
        [1, 0, 0, 0, 1, 0, 1, 0, 0, 0, 1],
        [1, 1, 1, 1, 1, 0, 1, 1, 1, 1, 1],
        [1, 4, 0, 0, 0, 0, 0, 0, 0, 4, 1],
        [1, 1, 1, 1, 1, 0, 1, 1, 1, 1, 1],
        [1, 0, 0, 0, 1, 0, 1, 0, 0, 1],
        [1, 0, 0, 0, 1, 0, 1, 0, 0, 1],
        [1, 0, 0, 0, 1, 0, 1, 0, 0, 1],
        [1, 0, 0, 0, 1, 2, 1, 0, 0, 0, 1],
        [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1]
      ],
      playerPos: { row: 1, col: 1 }
    }
  }
}

const startGame = async () => {
  gameStarted.value = true
  gameWon.value = false
  itemMessage.value = ''
  
  const { mapData, playerPos } = await loadMap()
  map.value = mapData
  player.x = playerPos.col
  player.y = playerPos.row
  
  nextTick(() => {
    const gamePlaying = document.querySelector('.game-playing')
    if (gamePlaying) {
      gamePlaying.focus()
    }
  })
}

const resetGame = async () => {
  gameWon.value = false
  itemMessage.value = ''
  player.weapon = ''
  player.armor = ''
  
  const { mapData, playerPos } = await loadMap()
  map.value = mapData
  player.x = playerPos.col
  player.y = playerPos.row
  
  nextTick(() => {
    const gamePlaying = document.querySelector('.game-playing')
    if (gamePlaying) {
      gamePlaying.focus()
    }
  })
}

const initMap = () => {
  // 读取地图文件
  const mapData = [
    [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
    [1, 0, 0, 0, 0, 1, 3, 0, 0, 0, 1],
    [1, 0, 0, 0, 0, 1, 0, 1, 0, 0, 1],
    [1, 0, 0, 0, 0, 1, 0, 1, 0, 0, 1],
    [1, 1, 1, 1, 1, 0, 1, 1, 1, 1, 1],
    [1, 4, 0, 0, 0, 0, 0, 0, 0, 4, 1],
    [1, 1, 1, 1, 1, 0, 1, 1, 1, 1, 1],
    [1, 0, 0, 0, 0, 1, 0, 1, 0, 0, 1],
    [1, 0, 0, 0, 0, 1, 0, 1, 0, 0, 1],
    [1, 0, 0, 0, 1, 2, 1, 0, 0, 0, 1],
    [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1]
  ]

  // 找到玩家位置
  let playerPos = { row: 1, col: 1 }
  for (let row = 0; row < 10; row++) {
    for (let col = 0; col < 10; col++) {
      if (mapData[row][col] === 2) {
        playerPos = { row, col }
        break
      }
    }
  }

  // 直接使用地图文件的数据
  map.value = mapData
  player.x = playerPos.col
  player.y = playerPos.row
}

const findAllNecessaryPaths = (map, positions) => {
  // 找到所有特殊点位之间的最短路径
  const allPaths = []
  
  for (let i = 0; i < positions.length; i++) {
    for (let j = i + 1; j < positions.length; j++) {
      const path = findShortestPath(map, positions[i], positions[j])
      if (path) {
        allPaths.push(path)
      }
    }
  }
  
  return allPaths
}

const findShortestPath = (map, start, end) => {
  // 使用BFS找到最短路径
  const queue = [[start]]
  const visited = new Set([`${start.row},${start.col}`])
  
  while (queue.length > 0) {
    const path = queue.shift()
    const current = path[path.length - 1]
    
    if (current.row === end.row && current.col === end.col) {
      return path
    }
    
    const directions = [[0, 1], [0, -1], [1, 0], [-1, 0]]
    for (const [dx, dy] of directions) {
      const newRow = current.row + dy
      const newCol = current.col + dx
      
      if (newRow >= 1 && newRow <= 8 && newCol >= 1 && newCol <= 8) {
        if (map[newRow][newCol] !== 1 && !visited.has(`${newRow},${newCol}`)) {
          visited.add(`${newRow},${newCol}`)
          queue.push([...path, { row: newRow, col: newCol }])
        }
      }
    }
  }
  
  return null
}

const checkConnectivity = (map, positions) => {
  const start = positions[0]
  const targets = positions.slice(1)
  
  const visited = new Set()
  const queue = [start]
  visited.add(`${start.row},${start.col}`)
  
  while (queue.length > 0) {
    const current = queue.shift()
    
    const directions = [[0, 1], [0, -1], [1, 0], [-1, 0]]
    for (const [dx, dy] of directions) {
      const newRow = current.row + dy
      const newCol = current.col + dx
      
      if (newRow >= 1 && newRow <= 8 && newCol >= 1 && newCol <= 8 && 
          map[newRow][newCol] !== 1 && !visited.has(`${newRow},${newCol}`)) {
        visited.add(`${newRow},${newCol}`)
        queue.push({ row: newRow, col: newCol })
      }
    }
  }
  
  return targets.every(pos => visited.has(`${pos.row},${pos.col}`))
}

const movePlayer = (dx, dy) => {
  if (gameWon.value) return

  const newX = player.x + dx
  const newY = player.y + dy

  // 检查是否越界或碰到墙壁
  if (newX < 0 || newX >= 10 || newY < 0 || newY >= 10) return
  if (map.value[newY][newX] === 1) return

  // 记录当前位置是否是宝箱
  const currentCell = map.value[player.y][player.x]
  const wasOnChest4 = currentCell === 4
  const wasOnChest5 = currentCell === 5

  // 清除当前位置的玩家
  map.value[player.y][player.x] = wasOnChest4 ? 4 : (wasOnChest5 ? 5 : 0)

  // 更新玩家位置
  player.x = newX
  player.y = newY

  // 检查是否到达目的地
  if (map.value[newY][newX] === 3) {
    gameWon.value = true
  }

  // 检查是否进入宝箱位置
  if (map.value[newY][newX] === 4) {
    if (!player.weapon && !player.armor) {
      // 随机获得武器或护甲
      if (Math.random() < 0.5) {
        itemMessage.value = '获得武器：铁剑'
      } else {
        itemMessage.value = '获得护甲：铠甲'
      }
    }
  }

  // 检查是否离开宝箱位置，获得物品
  if (wasOnChest && map.value[newY][newX] !== 4) {
    if (!player.weapon && !player.armor) {
      // 随机获得武器或护甲
      if (Math.random() < 0.5) {
        player.weapon = '铁剑'
      } else {
        player.armor = '铠甲'
      }
      itemMessage.value = ''
    }
  } else if (map.value[newY][newX] !== 4) {
    // 离开宝箱位置但没有获得物品（已经获得过了）
    itemMessage.value = ''
  }

  // 在新位置显示玩家
  map.value[newY][newX] = 2
}

const handleKeyDown = (e) => {
  const key = e.key.toLowerCase()
  switch (key) {
    case 'w':
      movePlayer(0, -1)
      break
    case 's':
      movePlayer(0, 1)
      break
    case 'a':
      movePlayer(-1, 0)
      break
    case 'd':
      movePlayer(1, 0)
      break
  }
}

const getCellIcon = (cell) => {
  switch (cell) {
    case 1: return '■' // 墙壁
    case 2: return '👤' // 玩家
    case 3: return '⬇' // 目的地（楼梯）
    case 4: return '📦' // 宝箱
    default: return ''
  }
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
  outline: none;
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
  position: relative;
}

.game-map {
  display: inline-block;
  border: 2px solid #000;
  margin-left: 10%;
}

.game-won {
  position: absolute;
  top: 20px;
  left: 50%;
  transform: translateX(-50%);
  font-size: 3rem;
  font-weight: bold;
  color: #4CAF50;
  text-align: center;
}

.reset-button-container {
  position: absolute;
  top: 80px;
  right: 20px;
}

.map-row {
  display: flex;
}

.map-cell {
  width: 50px;
  height: 50px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 24px;
  border: 1px solid #ddd;
}

.map-cell.wall {
  background: #333;
  color: #fff;
}

.map-cell.player {
  background: #4CAF50;
  color: #fff;
}

.map-cell.destination {
  background: #FFA726;
  color: #fff;
}

.item-message {
  position: absolute;
  bottom: 20px;
  left: 50%;
  transform: translateX(-50%);
  font-size: 1.2rem;
  font-weight: normal;
  color: #666;
  text-align: center;
  white-space: nowrap;
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
