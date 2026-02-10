<script setup lang="ts">
import { ref, onMounted } from 'vue';

const currentScene = ref(0);
const scenes = [
  'meteor-shower',
  'sky-tear',
  'grassland',
  'sea',
  'autumn-yunnan',
  'winter-northeast'
];

const enterScene = (sceneIndex: number) => {
  currentScene.value = sceneIndex;
};

onMounted(() => {
  // Auto transition between scenes every 10 seconds
  setInterval(() => {
    currentScene.value = (currentScene.value + 1) % scenes.length;
  }, 10000);
});
</script>

<template>
  <div class="visual-effects-container">
    <!-- Meteor Shower with Aurora -->
    <div 
      v-if="currentScene === 0" 
      class="scene meteor-shower"
    >
      <div class="aurora"></div>
      <div class="meteors">
        <div class="meteor" v-for="i in 20" :key="i"></div>
      </div>
    </div>

    <!-- Sky Tear Effect -->
    <div 
      v-else-if="currentScene === 1" 
      class="scene sky-tear"
    >
      <div class="sky"></div>
      <div class="tear">
        <div class="tear-line"></div>
        <div class="tear-line"></div>
        <div class="tear-line"></div>
      </div>
    </div>

    <!-- Grassland (Hulunbuir) -->
    <div 
      v-else-if="currentScene === 2" 
      class="scene grassland"
    >
      <div class="mountains"></div>
      <div class="grass">
        <div class="grass-blade" v-for="i in 50" :key="i"></div>
      </div>
    </div>

    <!-- Sea (Qingdao) -->
    <div 
      v-else-if="currentScene === 3" 
      class="scene sea"
    >
      <div class="sea-wave">
        <div class="wave" v-for="i in 5" :key="i"></div>
      </div>
    </div>

    <!-- Autumn in Yunnan -->
    <div 
      v-else-if="currentScene === 4" 
      class="scene autumn-yunnan"
    >
      <div class="autumn-trees">
        <div class="tree" v-for="i in 20" :key="i"></div>
      </div>
    </div>

    <!-- Winter in Northeast China -->
    <div 
      v-else-if="currentScene === 5" 
      class="scene winter-northeast"
    >
      <div class="snow-fall">
        <div class="snowflake" v-for="i in 100" :key="i"></div>
      </div>
      <div class="snow-landscape"></div>
    </div>

    <!-- Scene Navigation -->
    <div class="scene-navigation">
      <button 
        v-for="(scene, index) in scenes" 
        :key="scene"
        @click="enterScene(index)"
        :class="{ active: currentScene === index }"
      >
        {{ scene }}
      </button>
    </div>
  </div>
</template>

<style scoped>
.visual-effects-container {
  position: relative;
  width: 100vw;
  height: 100vh;
  overflow: hidden;
}

.scene {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  transition: opacity 1.5s ease-in-out;
  opacity: 0;
}

.scene:nth-child(1) {
  opacity: 1;
}

.scene.meteor-shower {
  background: linear-gradient(to bottom, #0a0a23, #1a1a4a);
  position: relative;
  overflow: hidden;
}

.aurora {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 60%;
  background: linear-gradient(
    to bottom,
    rgba(135, 206, 250, 0.3),
    rgba(138, 43, 226, 0.3),
    transparent
  );
  animation: aurora-move 20s ease-in-out infinite;
  z-index: 1;
}

.meteors {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 2;
}

.meteor {
  position: absolute;
  width: 2px;
  height: 100px;
  background: linear-gradient(
    to bottom,
    transparent,
    rgba(255, 255, 255, 0.8),
    rgba(255, 255, 255, 0.4),
    transparent
  );
  animation: meteor-fall 3s linear infinite;
  transform-origin: top center;
}

.meteor:nth-child(odd) {
  animation-delay: 1s;
}

.meteor:nth-child(3n) {
  animation-delay: 2s;
}

.meteor:nth-child(5n) {
  animation-delay: 0.5s;
}

@keyframes aurora-move {
  0%, 100% {
    transform: translateX(-10%) scale(1.1);
  }
  50% {
    transform: translateX(10%) scale(1.2);
  }
}

@keyframes meteor-fall {
  0% {
    top: -100px;
    left: calc(100% * var(--random-x, 0.5));
    transform: rotate(30deg) scale(0);
    opacity: 0;
  }
  10% {
    opacity: 1;
  }
  90% {
    opacity: 1;
  }
  100% {
    top: 100vh;
    left: calc(100% * var(--random-x, 0.5) - 500px);
    transform: rotate(30deg) scale(1);
    opacity: 0;
  }
}

.scene.sky-tear {
  background: linear-gradient(to bottom, #0a0a23, #1a1a4a);
  position: relative;
}

.sky {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: linear-gradient(
    to bottom,
    rgba(135, 206, 250, 0.2),
    rgba(138, 43, 226, 0.2),
    transparent
  );
}

.tear {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 300px;
  height: 300px;
  z-index: 10;
}

.tear-line {
  position: absolute;
  background: rgba(255, 255, 255, 0.8);
  box-shadow: 0 0 20px rgba(255, 255, 255, 0.8);
  animation: tear-expand 3s ease-in-out forwards;
}

.tear-line:nth-child(1) {
  top: 0;
  left: 50%;
  width: 2px;
  height: 100%;
  transform: translateX(-50%);
}

.tear-line:nth-child(2) {
  top: 50%;
  left: 0;
  width: 100%;
  height: 2px;
  transform: translateY(-50%);
}

.tear-line:nth-child(3) {
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  border: 2px solid rgba(255, 255, 255, 0.8);
  border-radius: 50%;
  box-shadow: 0 0 30px rgba(255, 255, 255, 0.8),
              inset 0 0 30px rgba(255, 255, 255, 0.3);
  animation: tear-circle-expand 3s ease-in-out forwards;
}

@keyframes tear-expand {
  0% {
    transform: scale(0);
    opacity: 0;
  }
  50% {
    opacity: 1;
  }
  100% {
    transform: scale(1);
    opacity: 0.8;
  }
}

@keyframes tear-circle-expand {
  0% {
    transform: scale(0);
    opacity: 0;
  }
  50% {
    opacity: 1;
  }
  100% {
    transform: scale(1.5);
    opacity: 0;
  }
}

.scene.grassland {
  background: linear-gradient(to bottom, #87CEEB, #98FB98);
  position: relative;
  overflow: hidden;
}

.mountains {
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  height: 40%;
  background: url('https://trae-api-cn.mchost.guru/api/ide/v1/text_to_image?prompt=Hulunbuir%20grassland%20mountains%20silhouette%20at%20sunset&image_size=landscape_16_9') center/cover no-repeat;
  z-index: 1;
}

.grass {
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  height: 60%;
  z-index: 2;
}

.grass-blade {
  position: absolute;
  bottom: 0;
  left: calc(100% * var(--random-x, 0.5));
  width: 4px;
  height: 80px;
  background: linear-gradient(to top, #228B22, #98FB98);
  border-radius: 2px 2px 0 0;
  animation: grass-sway 2s ease-in-out infinite;
  transform-origin: bottom center;
}

.grass-blade:nth-child(even) {
  animation-delay: 0.5s;
}

.grass-blade:nth-child(3n) {
  animation-delay: 1s;
}

.grass-blade:nth-child(5n) {
  height: 60px;
}

.grass-blade:nth-child(7n) {
  height: 100px;
}

@keyframes grass-sway {
  0%, 100% {
    transform: rotate(-5deg);
  }
  50% {
    transform: rotate(5deg);
  }
}

.scene.sea {
  background: linear-gradient(to bottom, #87CEEB, #1E90FF);
  position: relative;
  overflow: hidden;
}

.sea-wave {
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  height: 60%;
  background: url('https://trae-api-cn.mchost.guru/api/ide/v1/text_to_image?prompt=Qingdao%20sea%20coastline%20with%20waves%20and%20sandy%20beach&image_size=landscape_16_9') center/cover no-repeat;
}

.wave {
  position: absolute;
  bottom: 0;
  left: 0;
  width: 200%;
  height: 100px;
  background: rgba(30, 144, 255, 0.3);
  border-radius: 50% 50% 0 0;
  animation: wave-move 8s linear infinite;
}

.wave:nth-child(2) {
  animation-delay: 2s;
  height: 80px;
}

.wave:nth-child(3) {
  animation-delay: 4s;
  height: 60px;
}

.wave:nth-child(4) {
  animation-delay: 6s;
  height: 40px;
}

.wave:nth-child(5) {
  animation-delay: 8s;
  height: 20px;
}

@keyframes wave-move {
  0% {
    transform: translateX(0);
  }
  100% {
    transform: translateX(-50%);
  }
}

.scene.autumn-yunnan {
  background: linear-gradient(to bottom, #FFA07A, #FF6347);
  position: relative;
  overflow: hidden;
}

.autumn-trees {
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  height: 80%;
  background: url('https://trae-api-cn.mchost.guru/api/ide/v1/text_to_image?prompt=Yunnan%20autumn%20landscape%20with%20red%20maple%20trees%20and%20mountains&image_size=landscape_16_9') center/cover no-repeat;
}

.tree {
  position: absolute;
  bottom: 0;
  left: calc(100% * var(--random-x, 0.5));
  width: 60px;
  height: 120px;
  background: linear-gradient(to top, #8B4513, #CD853F);
  border-radius: 30px 30px 0 0;
  animation: tree-sway 3s ease-in-out infinite;
  transform-origin: bottom center;
}

.tree::before {
  content: '';
  position: absolute;
  top: -80px;
  left: -40px;
  width: 140px;
  height: 80px;
  background: linear-gradient(to bottom, #FF4500, #FF6347);
  border-radius: 50% 50% 0 0;
}

.tree:nth-child(even) {
  animation-delay: 1s;
}

.tree:nth-child(3n) {
  width: 40px;
  height: 90px;
}

.tree:nth-child(3n)::before {
  width: 100px;
  height: 60px;
  top: -60px;
  left: -30px;
}

.tree:nth-child(5n) {
  width: 80px;
  height: 150px;
}

.tree:nth-child(5n)::before {
  width: 180px;
  height: 100px;
  top: -100px;
  left: -50px;
}

@keyframes tree-sway {
  0%, 100% {
    transform: rotate(-2deg);
  }
  50% {
    transform: rotate(2deg);
  }
}

.scene.winter-northeast {
  background: linear-gradient(to bottom, #E0FFFF, #B0C4DE);
  position: relative;
  overflow: hidden;
}

.snow-fall {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 1;
}

.snowflake {
  position: absolute;
  top: -10px;
  left: calc(100% * var(--random-x, 0.5));
  width: 10px;
  height: 10px;
  background: white;
  border-radius: 50%;
  animation: snow-fall linear infinite;
  opacity: 0.8;
}

.snowflake:nth-child(even) {
  animation-duration: 8s;
}

.snowflake:nth-child(3n) {
  animation-duration: 10s;
}

.snowflake:nth-child(5n) {
  animation-duration: 12s;
}

.snowflake:nth-child(7n) {
  width: 15px;
  height: 15px;
}

.snowflake:nth-child(11n) {
  width: 5px;
  height: 5px;
}

.snow-landscape {
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  height: 60%;
  background: url('https://trae-api-cn.mchost.guru/api/ide/v1/text_to_image?prompt=Northeast%20China%20winter%20landscape%20with%20snow-covered%20trees%20and%20mountains&image_size=landscape_16_9') center/cover no-repeat;
  z-index: 2;
}

@keyframes snow-fall {
  0% {
    top: -10px;
    transform: translateX(0) rotate(0deg);
  }
  100% {
    top: 100vh;
    transform: translateX(100px) rotate(360deg);
  }
}

.scene-navigation {
  position: fixed;
  bottom: 20px;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  gap: 10px;
  z-index: 100;
}

.scene-navigation button {
  padding: 8px 16px;
  background: rgba(0, 0, 0, 0.5);
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 12px;
  transition: all 0.3s ease;
}

.scene-navigation button:hover {
  background: rgba(0, 0, 0, 0.7);
}

.scene-navigation button.active {
  background: rgba(255, 255, 255, 0.8);
  color: black;
}

/* Scene transition animation */
.scene {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  opacity: 0;
  transition: opacity 1.5s ease-in-out;
}

.scene:nth-child(1 + v-if="currentScene === 0") {
  opacity: 1;
}

.scene:nth-child(2 + v-else-if="currentScene === 1") {
  opacity: 1;
}

.scene:nth-child(3 + v-else-if="currentScene === 2") {
  opacity: 1;
}

.scene:nth-child(4 + v-else-if="currentScene === 3") {
  opacity: 1;
}

.scene:nth-child(5 + v-else-if="currentScene === 4") {
  opacity: 1;
}

.scene:nth-child(6 + v-else-if="currentScene === 5") {
  opacity: 1;
}
</style>