<template>
  <div>
    <h1>分數: {{ score }}</h1>
    <div
      id="game-area"
      style="
        position: relative;
        width: 600px;
        height: 400px;
        border: 2px solid black;
        background: #eef;
      "
    >
      <!-- 小人 -->
      <div
        id="player"
        :style="{
          position: 'absolute',
          bottom: '10px',
          left: playerX + 'px',
          width: playerWidth + 'px',
          height: playerHeight + 'px',
          background: 'red',
        }"
      ></div>

      <!-- 掉落寶物 -->
      <div
        v-for="(item, index) in items"
        :key="index"
        :style="{
          position: 'absolute',
          top: item.y + 'px',
          left: item.x + 'px',
          width: itemSize + 'px',
          height: itemSize + 'px',
          background: 'gold',
          borderRadius: '50%',
        }"
      ></div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";

// 遊戲設定
const gameWidth = 600;
const gameHeight = 400;
const playerWidth = 50;
const playerHeight = 50;
const itemSize = 20;

const score = ref(0);
const playerX = ref(gameWidth / 2 - playerWidth / 2);
const playerY = gameHeight - playerHeight - 10;

// 掉落寶物陣列
const items = ref([]);

// 隨機生成寶物
function spawnItem() {
  items.value.push({
    x: Math.random() * (gameWidth - itemSize),
    y: 0,
  });
}

// 移動寶物
function moveItems() {
  items.value.forEach((item, index) => {
    item.y += 5;

    // 碰到小人就加分
    if (
      item.y + itemSize >= playerY &&
      item.x + itemSize >= playerX.value &&
      item.x <= playerX.value + playerWidth
    ) {
      score.value += 1;
      items.value.splice(index, 1);
    }

    // 超出遊戲區域就移除
    if (item.y > gameHeight) {
      items.value.splice(index, 1);
    }
  });
}

// 鍵盤控制小人
function handleKey(e) {
  const step = 20;
  if (e.key === "ArrowLeft") playerX.value = Math.max(0, playerX.value - step);
  if (e.key === "ArrowRight")
    playerX.value = Math.min(gameWidth - playerWidth, playerX.value + step);
}

onMounted(() => {
  window.addEventListener("keydown", handleKey);

  // 每 1 秒生成一個寶物
  setInterval(spawnItem, 1000);

  // 遊戲循環
  function gameLoop() {
    moveItems();
    requestAnimationFrame(gameLoop);
  }
  gameLoop();
});
</script>
