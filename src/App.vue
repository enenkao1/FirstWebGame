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
  floor: 0,
  hp: 100,
  maxHp: 100,
  status: '正常',
  weapon: '',
  armor: '',
  x: 1,
  y: 1
})

const generateMap = () => {
  const size = 11;
  const numSpecialPoints = 3; 
  const minDeadEnds = numSpecialPoints + 1;
  
  let mapData;
  let deadEnds;
  let candidates;
  let attempts = 0;

  while (attempts < 100) {
    attempts++;
    mapData = Array.from({ length: size }, () => Array(size).fill(1));

    // 1. 迷宫生成 (递归回溯)
  const startNodes = [{ r: 1, c: 1 }, { r: 1, c: 9 }, { r: 9, c: 1 }, { r: 9, c: 9 }];
  const startNode = startNodes[Math.floor(Math.random() * startNodes.length)];
  const visited = new Set();
  const stack = [startNode];
  visited.add(`${startNode.r},${startNode.c}`);
  mapData[startNode.r][startNode.c] = 0;

  while (stack.length > 0) {
    const curr = stack[stack.length - 1];
    const neighbors = [];
    const dirs = [[0, 2], [0, -2], [2, 0], [-2, 0]];

    for (const [dr, dc] of dirs) {
      const nr = curr.r + dr;
      const nc = curr.c + dc;
      if (nr >= 1 && nr <= 9 && nc >= 1 && nc <= 9 && !visited.has(`${nr},${nc}`)) {
        neighbors.push({ r: nr, c: nc, dr: dr / 2, dc: dc / 2 });
      }
    }

    if (neighbors.length > 0) {
      const next = neighbors[Math.floor(Math.random() * neighbors.length)];
      mapData[next.r][next.c] = 0;
      mapData[curr.r + next.dr][curr.c + next.dc] = 0;
      visited.add(`${next.r},${next.c}`);
      stack.push({ r: next.r, c: next.c });
    } else {
      stack.pop();
    }
  }

  // 2. 寻找所有候选点位（通路格）并评分
  candidates = [];
  deadEnds = [];
  for (let r = 1; r <= 9; r += 2) {
    for (let c = 1; c <= 9; c += 2) {
      if (mapData[r][c] === 0) {
        let walls = 0;
        // 检查上下左右四个邻居
        if (mapData[r-1][c] === 1) walls++;
        if (mapData[r+1][c] === 1) walls++;
        if (mapData[r][c-1] === 1) walls++;
        if (mapData[r][c+1] === 1) walls++;
        
        const cand = { 
          r, c, walls, 
          score: walls * 10 + ((r === 1 || r === 9) && (c === 1 || c === 9) ? 5 : 0) 
        };
        candidates.push(cand);
        if (walls >= 3) deadEnds.push(cand);
      }
    }
  }

  if (deadEnds.length >= minDeadEnds) break;
}

  // 3. 选取点位
  // 随机打乱死胡同列表，实现随机分配
  for (let i = deadEnds.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [deadEnds[i], deadEnds[j]] = [deadEnds[j], deadEnds[i]];
  }

  // 玩家、楼梯和宝箱直接分配到前三个随机死胡同里
  const pPos = deadEnds[0];
  const sPos = deadEnds[1];
  const cPos = deadEnds[2];

  mapData[pPos.r][pPos.c] = 2;
  mapData[sPos.r][sPos.c] = 3;
  mapData[cPos.r][cPos.c] = 4;

  return { mapData, playerPos: { row: pPos.r, col: pPos.c } };
}

const startGame = async () => {
  gameStarted.value = true
  gameWon.value = false
  itemMessage.value = ''
  
  const { mapData, playerPos } = generateMap()
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
  player.floor = 0
  
  const { mapData, playerPos } = generateMap()
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

const movePlayer = (dx, dy) => {
  if (gameWon.value) return

  const newX = player.x + dx
  const newY = player.y + dy

  // 检查是否越界或碰到墙壁 (11x11 边界)
  if (newX < 0 || newX >= 11 || newY < 0 || newY >= 11) return
  if (map.value[newY][newX] === 1) return

  // 记录当前位置是否是宝箱
  const currentCell = map.value[player.y][player.x]
  const wasOnChest = currentCell === 4

  // 清除当前位置的玩家
  map.value[player.y][player.x] = wasOnChest ? 4 : 0

  // 检查是否进入宝箱位置
  if (map.value[newY][newX] === 4) {
    // 规则1：既没有武器也没有护甲时，随机获得
    if (!player.weapon && !player.armor) {
      if (Math.random() < 0.5) {
        itemMessage.value = '获得武器：铁剑'
        player.weapon = '铁剑'
      } else {
        itemMessage.value = '获得护甲：铠甲'
        player.armor = '铠甲'
      }
    }
    // 规则2：有铁剑，获得铠甲
    else if (player.weapon && !player.armor) {
      itemMessage.value = '获得护甲：铠甲'
      player.armor = '铠甲'
    }
    // 规则3：有铠甲，获得铁剑
    else if (!player.weapon && player.armor) {
      itemMessage.value = '获得武器：铁剑'
      player.weapon = '铁剑'
    }
    // 规则4：都有装备，随机获得一个
    else if (player.weapon && player.armor) {
      if (Math.random() < 0.5) {
        itemMessage.value = '获得武器：铁剑'
      } else {
        itemMessage.value = '获得护甲：铠甲'
      }
    }
  } else {
    // 离开宝箱位置，清除提示
    itemMessage.value = ''
  }

  // 检查是否到达目的地 (楼梯)
  if (map.value[newY][newX] === 3) {
    if (player.floor <= -10) {
      gameWon.value = true
    } else {
      player.floor -= 1
      const { mapData, playerPos: pPos } = generateMap()
      map.value = mapData
      player.x = pPos.col
      player.y = pPos.row
      itemMessage.value = `进入地下第 ${Math.abs(player.floor)} 层`
      return // 切换地图后直接返回
    }
  }

  // 更新玩家位置
  player.x = newX
  player.y = newY

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

