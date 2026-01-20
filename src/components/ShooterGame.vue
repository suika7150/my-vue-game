<template>
  <div>
    <h2>分數：{{ score }}</h2>

    <div
      style="
        position: relative;
        width: 600px;
        height: 400px;
        border: 2px solid black;
        background: #eef;
        overflow: hidden;
      "
    >
      <!-- 角色 -->
      <div
        :style="{
          position: 'absolute',
          bottom: '10px',
          left: playerX + 'px',
          width: '40px',
          height: '40px',
          background: 'blue',
        }"
      ></div>

      <!-- 子彈 -->
      <div
        v-for="(b, i) in bullets"
        :key="'b' + i"
        :style="{
          position: 'absolute',
          left: b.x + 'px',
          top: b.y + 'px',
          width: '6px',
          height: '12px',
          background: 'black',
        }"
      ></div>

      <!-- 怪 -->
      <div
        v-for="(m, i) in monsters"
        :key="'m' + i"
        :style="{
          position: 'absolute',
          left: m.x + 'px',
          top: m.y + 'px',
          width: '40px',
          height: '40px',
          background: 'red',
        }"
      ></div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";

const gameWidth = 600;
const gameHeight = 400;

const playerX = ref(gameWidth / 2 - 20);
const score = ref(0);

const bullets = ref([]);
const monsters = ref([]);

// 鍵盤控制
function handleKey(e) {
  if (e.key === "ArrowLeft") {
    playerX.value = Math.max(0, playerX.value - 20);
  }
  if (e.key === "ArrowRight") {
    playerX.value = Math.min(gameWidth - 40, playerX.value + 20);
  }
  if (e.key === " ") {
    bullets.value.push({
      x: playerX.value + 17,
      y: gameHeight - 60,
    });
  }
}

// 生成怪
function spawnMonster() {
  monsters.value.push({
    x: Math.random() * (gameWidth - 40),
    y: 0,
  });
}

// 碰撞判定
function hit(a, b, size = 40) {
  return (
    a.x < b.x + size && a.x + 10 > b.x && a.y < b.y + size && a.y + 10 > b.y
  );
}

// 遊戲主迴圈
function gameLoop() {
  // 子彈往上
  bullets.value.forEach((b, bi) => {
    b.y -= 8;
    if (b.y < 0) bullets.value.splice(bi, 1);
  });

  // 怪往下
  monsters.value.forEach((m, mi) => {
    m.y += 2;
    if (m.y > gameHeight) monsters.value.splice(mi, 1);
  });

  // 碰撞檢查
  bullets.value.forEach((b, bi) => {
    monsters.value.forEach((m, mi) => {
      if (hit(b, m)) {
        bullets.value.splice(bi, 1);
        monsters.value.splice(mi, 1);
        score.value++;
      }
    });
  });

  requestAnimationFrame(gameLoop);
}

onMounted(() => {
  window.addEventListener("keydown", handleKey);
  setInterval(spawnMonster, 1000);
  gameLoop();
});
</script>
