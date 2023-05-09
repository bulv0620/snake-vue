<template>
  <div class="container">
    <div class="stage">
      <div class="row" v-for="(col, y) in stage" :key="y">
        <div class="col" :class="{
          wall: item.type === 'wall',
          snake: item.type === 'snake',
          food: item.type === 'food',
        }" v-for="(item, x) in col" :key="x"></div>
      </div>
    </div>
    <p class="status" @click="init" v-if="gameOver">游戏结束，得分{{ score }}，点击这里重来。</p>
    <p class="status" @click="startGame" v-else-if="pause">游戏已暂停，点击这里继续</p>
    <p class="status" @click="pauseGame" v-else>键盘↑↓←→控制，Z加速，X减速，当前速度{{ speed }}，点击这里暂停</p>
  </div>
</template>

<script setup>
import { ref, computed, onBeforeMount, onMounted } from "vue";

// 移动Interval
let moveInterval = null;
// 移动方向
let direction = "left";
// 步控器
let step = [];
// 速度
let speed = ref(120);
// 行数 y
const row = 40;
// 列数 x
const col = 80;
// 游戏结束
const gameOver = ref(false);
// 游戏暂停状态
const pause = ref(false);
// 分数
const score = ref(0);
// 地图
const map = ref([]);
// 蛇蛇
const snake = ref([]);
// 食物
const food = ref([]);
// 场景
const stage = computed(() => {
  const renderStage = JSON.parse(JSON.stringify(map.value));
  for (const snakeBody of snake.value) {
    renderStage[snakeBody.y][snakeBody.x].type = "snake";
  }
  for (const foodItem of food.value) {
    renderStage[foodItem.y][foodItem.x].type = "food";
  }
  return renderStage;
});

onBeforeMount(init);
onMounted(() => {
  window.onblur = (e) => {
    pauseGame()
  }
  window.document.onkeydown = (e) => {
    const keyNum = window.event ? e.keyCode : e.which;
    // console.log(keyNum);
    switch (keyNum) {
      case 37:
      case 65:
        if (!['left', 'right'].includes(direction)) {
          direction = 'left'
          step.push('left')
        }
        break;
      case 38:
      case 87:
        if (!['top', 'bottom'].includes(direction)) {
          direction = 'top'
          step.push('top')
        }
        break;
      case 39:
      case 68:
        if (!['left', 'right'].includes(direction)) {
          direction = 'right'
          step.push('right')
        }
        break;
      case 40:
      case 83:
        if (!['top', 'bottom'].includes(direction)) {
          direction = 'bottom'
          step.push('bottom')
        }
        break;
      case 90:
        // 加速
        if (speed.value > 40) {
          speed.value -= 20
          startGame()
        }
        break;
      case 88:
        // 减速
        if (speed.value < 220) {
          speed.value += 20
          startGame()
        }
        break;
      default:
        break;
    }
  };
});

function init() {
  map.value = new Array(row).fill(null).map((_, y) =>
    new Array(col).fill(null).map((_, x) => ({
      x,
      y,
      type: x === 0 || x === col - 1 || y === 0 || y === row - 1 ? "wall" : "",
    }))
  );

  direction = 'left'
  step = []
  gameOver.value = false
  score.value = 0
  snake.value = initSnake();
  food.value = []
  food.value.push(createFood(snake.value));

  startGame()
}

function startGame() {
  pause.value = false
  clearInterval(moveInterval)
  moveInterval = setInterval(moveAction, speed.value);
}

function pauseGame() {
  pause.value = true
  clearInterval(moveInterval)
}

function initSnake() {
  const snakeDefaultValue = [
    {
      x: parseInt(col / 2) + 1,
      y: parseInt(row / 2),
    },
    {
      x: parseInt(col / 2),
      y: parseInt(row / 2),
    },
    {
      x: parseInt(col / 2) - 1,
      y: parseInt(row / 2),
    },
  ];
  return snakeDefaultValue;
}

function createFood(snake) {
  let randonX = Math.floor(Math.random() * (col - 2)) + 1;
  let randonY = Math.floor(Math.random() * (row - 2)) + 1;

  while (snake.find((el) => el.x === randonX && el.y === randonY)) {
    randonX = Math.floor(Math.random() * (col - 2)) + 1;
    randonY = Math.floor(Math.random() * (row - 2)) + 1;
  }
  return {
    x: randonX,
    y: randonY,
  };
}

function moveAction() {
  // 头部添加
  const currentHead = snake.value[snake.value.length - 1];
  const newHead = {
    x: currentHead.x,
    y: currentHead.y,
  };
  const d = step.length === 0 ? direction : step.shift()
  switch (d) {
    case "left":
      newHead.x = newHead.x - 1;
      break;
    case "right":
      newHead.x = newHead.x + 1;
      break;
    case "top":
      newHead.y = newHead.y - 1;
      break;
    case "bottom":
      newHead.y = newHead.y + 1;
      break;
    default:
      break;
  }

  const foodIndex = food.value.findIndex(
    (el) => el.x === newHead.x && el.y === newHead.y
  );
  if (foodIndex < 0) {
    // 正常移动
    snake.value.shift();
  } else {
    // 食物被吃
    food.value.splice(foodIndex, 1);
    food.value.push(createFood(snake.value));
    score.value++;
  }

  // 撞墙
  if (["wall", "snake"].includes(stage.value[newHead.y][newHead.x].type)) {
    clearInterval(moveInterval);
    gameOver.value = true;
  }

  snake.value.push(newHead);
}
</script>

<style scoped lang="less">
.container {
  width: 100vw;
  height: 100vh;
  background: #000;

  .stage {
    margin: 0 auto;
    width: 800px;
    height: 400px;
    // background: #fff;

    .row {
      height: 10px;
      // background: grey;
      display: flex;

      .col {
        width: 10px;
        height: 10px;
        // background: #fff;

        &.wall {
          background: grey;
        }

        &.food {
          background: #fff;
        }

        &.snake {
          background: seagreen;
        }
      }
    }
  }

  .status {
    color: white;
    text-align: center;
    cursor: pointer;
  }
}
</style>
