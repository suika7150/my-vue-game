<template>
  <div>
    <h2>分數：{{ score }}</h2>
  </div>
  <div class="game-container">
    <div class="stars"></div>

    <!-- 星球 -->

    <div
      v-for="d in decorations"
      :key="d.id"
      :style="{
        position: 'absolute',
        left: d.x + 'px',
        top: d.y + 'px',
        width: d.size + 'px',
        height: d.size + 'px',
        backgroundImage: `url(${planetImg})`,
        backgroundSize: 'contain',
        backgroundRepeat: 'no-repeat',
        zIndex: 0,
        pointerEvents: 'none',
      }"
    ></div>

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

  <!-- 遊戲內容 -->
</template>

<script setup>
import { ref, onMounted } from "vue";
// import planetGIF from "/planet.gif";
import planetImg from "/planet.png";

const gameWidth = 600;
const gameHeight = 400;

const playerX = ref(gameWidth / 2 - 20);
const score = ref(0);

const bullets = ref([]);
const monsters = ref([]);

const leftPressed = ref(false);
const rightPressed = ref(false);

const shootPressed = ref(false);

const decorations = ref([]);

const speed = 4;
let lastShot = 0;

// 鍵盤控制
function handleKeyDown(e) {
  if (e.key === "ArrowLeft") leftPressed.value = true;
  if (e.key === "ArrowRight") rightPressed.value = true;
  if (e.key === " ") shootPressed.value = true;
}

function handleKeyUp(e) {
  if (e.key === "ArrowLeft") leftPressed.value = false;
  if (e.key === "ArrowRight") rightPressed.value = false;
  if (e.key === " ") shootPressed.value = false;
}

//星球
function spawnPlanet() {
  decorations.value.push({
    id: Date.now(),
    x: Math.random() * (gameWidth - 100),
    y: -150,
    size: 80 + Math.random() * 70,
    speed: 0.5 + Math.random() * 0.5,
  });
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
  // 角色移動
  if (leftPressed.value) {
    playerX.value = Math.max(0, playerX.value - speed);
  }
  if (rightPressed.value) {
    playerX.value = Math.min(gameWidth - 40, playerX.value + speed);
  }

  //射擊
  if (shootPressed.value && Date.now() - lastShot > 250) {
    bullets.value.push({
      x: playerX.value + 17,
      y: gameHeight - 60,
    });
    lastShot = Date.now();
  }

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

  // 星球移動邏輯
  decorations.value.forEach((d, i) => {
    d.y += d.speed;
    if (d.y > gameHeight + 100) {
      decorations.value.splice(i, 1);
    }
  });

  requestAnimationFrame(gameLoop);
}

onMounted(() => {
  window.addEventListener("keydown", handleKeyDown);
  window.addEventListener("keyup", handleKeyUp);
  setInterval(spawnMonster, 1000);
  setInterval(spawnPlanet, 8000);
  gameLoop();
});
</script>

<style scoped>
/* 遊戲主容器 */
.game-container {
  position: relative;
  width: 1000px;
  height: 500px;
  /* padding-bottom: 100px; */
  border: 2px solid black;
  background: white; /* 深邃宇宙藍 */
  /* overflow: hidden; */
}

/* 用 CSS 生成星星 */
.stars {
  position: absolute;
  background: #00051a;
  width: 1px;
  height: 1px;
  background: transparent;
  /* z-index: 100; */
  /* 這裡生成大量星星陰影：x位置, y位置, 顏色 */
  box-shadow:
    50px 30px #ffffff,
    120px 80px #fff,
    200px 50px #fff,
    300px 150px #fff,
    450px 20px #fff,
    550px 300px #fff,
    80px 350px #fff,
    400px 380px #fff,
    250px 280px #fff,
    150px 200px #fff,
    500px 100px #fff,
    30px 120px #fff;
  /* 如果想要更多星星，可以繼續加下去 */

  animation: starsScroll 20s linear infinite;
}

/* 讓星星動起來 */
/* @keyframes starsScroll {
  from {
    transform: translateY(0px);
  }
  to {
    transform: translateY(400px);
  }
} */
</style>
