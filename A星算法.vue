<script setup>
import { ref, onMounted } from 'vue'
// 连线避开障碍 A*星算法
//考试题目：
//在8x8生成的随机地图上，请让方块以最短路程走向「绿色」终点。
// 要求：
// 1. 不能穿过黑色的方块。（5%）
// 2. 要以最短路程完成。（5%）
// 3. 每次移动要有合适的停顿。（2%）
// 4. 到达终点后要显示走过的格子数。“本轮路程：3格”（5%）
// 5. 代码优雅度（5%）

const grids = ref([])

// 生成指定范围内的随机整数
function getRandomInt(min, max) {
  min = Math.ceil(min)
  max = Math.floor(max)
  return Math.floor(Math.random() * (max - min + 1)) + min
}

//——————————————————————————————————————————————
const pathLength = ref(0)
const showPath = ref(false)
const ROWS = 8
const COLS = 8
// 定义四个方向的移动（上、右、下、左）
const directions = [
  { x: -1, y: 0 }, // 上
  { x: 0, y: 1 }, // 右
  { x: 1, y: 0 }, // 下
  { x: 0, y: -1 } // 左
]

//————————————————————————————————————————————————

// 初始化网格，设置障碍、起点和终点
const getGrid = function (x, y) {
  for (let x = 0; x < 8; x++) {
    let row = []
    for (let y = 0; y < 8; y++) {
      row.push({ x, y, color: 'white' })
    }
    grids.value.push(row)
  }

  //———————————————————————————————————————————————
  // 随机放置起点（绿色）和终点（红色）
  let startX = ROWS - 1
  let startY = getRandomInt(0, COLS - 1)
  let endX = 0
  let endY = getRandomInt(0, COLS - 1)

  grids.value[7][startY].color = 'red'
  grids.value[0][endY].color = 'green'

  // 放置随机障碍（黑色方块）
  makeBlock(8)

  // 运行A*算法寻找最短路径
  let path = aStar(startX, startY, endX, endY)
  if (path.length > 0) {
    showPath.value = true
    pathLength.value = path.length
    animatePath(path)
  }
  //—————————————————————————————————————————————————
}

// 随机放置黑色障碍
const makeBlock = function (count) {
  var list = []
  while (list.length < count) {
    let randx = getRandomInt(1, grids.value.length - 2)
    let randy = getRandomInt(1, grids.value.length - 2)

    if (!list.find((item) => item.x == randx && item.y == randy)) {
      grids.value[randx][randy].color = 'black'
      list.push({ x: randx, y: randy })
    }
  }
}

//—————————————————————————————————————————
// A*算法实现
function aStar(startX, startY, endX, endY) {
  let openList = []
  let closedList = []

  // 初始化起点
  let startNode = grids.value[startX][startY]
  startNode.g = 0
  startNode.f = heuristic(startX, startY, endX, endY)
  openList.push(startNode)

  while (openList.length > 0) {
    // 获取openList中f值最小的节点作为当前节点
    let currentNode = openList.reduce(
      (minNode, node) => (node.f < minNode.f ? node : minNode),
      openList[0]
    )

    // 将当前节点从openList移到closedList
    openList = openList.filter((node) => !(node.x === currentNode.x && node.y === currentNode.y))
    closedList.push(currentNode)

    // 如果到达终点节点，重构路径并返回
    if (currentNode.x === endX && currentNode.y === endY) {
      return reconstructPath(currentNode)
    }

    // 探索当前节点的邻居节点
    for (let direction of directions) {
      let neighborX = currentNode.x + direction.x
      let neighborY = currentNode.y + direction.y

      if (isValidLocation(neighborX, neighborY) && !isInList(closedList, neighborX, neighborY)) {
        let neighborNode = grids.value[neighborX][neighborY]
        let tentativeG = currentNode.g + 1 // 假设每一步代价为1

        if (!isInList(openList, neighborX, neighborY) || tentativeG < neighborNode.g) {
          neighborNode.parent = currentNode
          neighborNode.g = tentativeG
          neighborNode.f = neighborNode.g + heuristic(neighborX, neighborY, endX, endY)

          if (!isInList(openList, neighborX, neighborY)) {
            openList.push(neighborNode)
          }
        }
      }
    }
  }

  // 如果openList为空且未找到终点节点，返回空路径
  return []
}

// 启发函数（曼哈顿距离）
function heuristic(x1, y1, x2, y2) {
  return Math.abs(x1 - x2) + Math.abs(y1 - y2)
}

// 检查位置是否在网格范围内且不是障碍物
function isValidLocation(x, y) {
  return x >= 0 && x < ROWS && y >= 0 && y < COLS && grids.value[x][y].color !== 'black'
}

// 检查坐标为(x, y)的节点是否在列表中
function isInList(list, x, y) {
  return list.some((node) => node.x === x && node.y === y)
}

// 从终点节点重构路径到起点节点
function reconstructPath(endNode) {
  let path = []
  let currentNode = endNode
  while (currentNode.parent) {
    path.unshift(currentNode)
    currentNode = currentNode.parent
  }
  // 将起点节点添加到路径中
  path.unshift(grids.value[currentNode.x][currentNode.y])
  return path
}
// 动画展示路径
function animatePath(path) {
  let animatedPath = path.slice() // 复制路径以进行动画
  let i = 0
  let interval = setInterval(() => {
    if (i < animatedPath.length) {
      let node = grids.value[animatedPath[i].x][animatedPath[i].y]
      if (node.color !== 'green' && node.color !== 'red') {
        node.color = 'blue' // 在动画中临时标记为蓝色
      }
      i++
    } else {
      clearInterval(interval)
      // 动画完成后更新网格状态
      // 将路径上的节点保持为蓝色
    }
  }, 200) // 根据需要调整间隔
}

// 在需要清除路径时调用该函数
function clearPath() {
  for (let r = 0; r < ROWS; r++) {
    for (let c = 0; c < COLS; c++) {
      if (grids.value[r][c].color === 'blue') {
        grids.value[r][c].color = 'white' // 将路径上的节点重新设置为白色
      }
    }
  }
}

//———————————————————————————————————————

onMounted(() => {
  getGrid()
})
</script>
<template>
  <div class="panel">
    <div class="row" v-for="(row, rowIndex) in grids" :key="rowIndex">
      <div
        :class="['tile', item.color]"
        v-for="(item, colIndex) in row"
        :key="`${rowIndex}-${colIndex}`"
      >
        {{ item.x }},{{ item.y }}
      </div>
    </div>
    <div class="info">
      <p v-if="showPath">本轮路程: {{ pathLength }} 格</p>
    </div>
  </div>
</template>
<style scoped>
.panel {
  width: 100vw;
  height: 100vh;
  background-color: #938888;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  gap: 4px;
  padding: 20px;
}

.tile {
  width: 50px;
  height: 50px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.row {
  display: flex;
  gap: 4px;
}

.white {
  background-color: white;
}

.red {
  background-color: red;
}

.green {
  background-color: green;
}

.black {
  color: aliceblue;
  background-color: black;
}

.blue {
  background-color: palegreen;
}
</style>