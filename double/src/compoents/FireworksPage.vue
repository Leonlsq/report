<template>
  <div class="cinema-container" @click="startShow">
    <canvas ref="canvasRef"></canvas>
    
    <transition name="fade">
      <div v-if="!hasStarted" class="intro-overlay">
        <div class="intro-text-wrapper">
          <p class="intro-text">轻触屏幕</p>
        </div>
      </div>
    </transition>

    <div v-if="hasStarted" class="overlay-content">
      <div class="text-wrapper">
        <h1 class="title handwritten-title" :class="{ 'start-typing': showCenterTitle }">
          Happy Birthday, My Lovely Ty
        </h1>

        <div class="subtitle-container" :class="{ 'visible': showCenterSubtitle }">
          <p class="subtitle">愿我们的爱，如这漫天烟火，绚烂且长明。</p>
          <p class="date">2025.11.28</p> 
        </div>
      </div>
    </div>

    <transition name="fade-slow">
      <div v-if="showBottomText" class="bottom-text-overlay">
        <p class="new-chinese-text">相遇已是上上签，再相遇便是一辈子。</p>
      </div>
    </transition>

    <transition name="fade-slow">
      <div v-if="showFinalTextContainer" class="final-text-left">
        <p>
          {{ displayedFinalText }}
          <span v-if="isFinalTextTyping" class="cursor">|</span>
        </p>
      </div>
    </transition>

  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue';

// --- 核心变量 ---
const canvasRef = ref<HTMLCanvasElement | null>(null);
let ctx: CanvasRenderingContext2D | null = null;
let width = 0;
let height = 0;
let animationId: number;

// 状态
const hasStarted = ref(false);
const isRandomFireworksActive = ref(false);

// 文字显示控制状态
const showCenterTitle = ref(false);    
const showFinalTextContainer = ref(false); 
const showCenterSubtitle = ref(false); 
const showBottomText = ref(false);     

// 左侧打字机相关
const displayedFinalText = ref(''); 
const fullFinalText = "这场烟花不仅是我数十小时的工作，也是我们爱的见证。";
const isFinalTextTyping = ref(false);
const TYPE_SPEED = 160; // 打字速度

// --- ⭐ 动态时间轴控制中心 (优化版) ⭐ ---
const startShow = () => {
  if (!hasStarted.value) {
    hasStarted.value = true;
    isRandomFireworksActive.value = true; 

    // 1. T+0s: 立即显示主标题 (CSS动画耗时4s)
    showCenterTitle.value = true;

    // 2. T+6s: 【修改点】给主标题留出2秒阅读缓冲，再开始左侧独白
    // 原来是4000，现在改为6000
    const delayLeftText = 7000; 
    setTimeout(() => {
      showFinalTextContainer.value = true;
      startTypewriter();
    }, delayLeftText);

    // --- 动态计算后续时间点 (自动顺延) ---
    
    const typingDuration = fullFinalText.length * TYPE_SPEED;
    const readBuffer = 3000; // 读完后的留白
    
    // 3. 中间副标题 ("愿我们的爱...")
    const delaySubtitle = delayLeftText + typingDuration + readBuffer;
    setTimeout(() => {
      showCenterSubtitle.value = true;
    }, delaySubtitle);

    // 4. 底部文字 ("相遇...")
    // 这里的间隔也稍微拉大一点 (4000 -> 4500)，节奏更舒缓
    const delayBottom = delaySubtitle + 4000;
    setTimeout(() => {
      showBottomText.value = true;
    }, delayBottom);

    // 5. 暂停随机烟花 (清场)
    const delayClear = delayBottom + 5000;
    setTimeout(() => {
      isRandomFireworksActive.value = false;
    }, delayClear);

    // 6. 发射爱心烟花
    const delayHeart = delayClear + 1000;
    setTimeout(() => {
      launchHeartPattern();
    }, delayHeart);

    // 7. 恢复随机烟花
    setTimeout(() => {
      isRandomFireworksActive.value = true;
    }, delayHeart + 6000);
  }
};

// --- 爱心烟花阵列 ---
const launchHeartPattern = () => {
  if (!ctx) return;
  const centerX = width / 2;
  const centerY = height * 0.38; 
  const scale = Math.min(width, height) * 0.016; 
  const totalPoints = 40; 
  const steps = totalPoints / 2; 
  
  for (let i = 0; i <= steps; i++) {
    const stepSize = Math.PI / steps;
    const t1 = Math.PI - (i * stepSize);
    const t2 = Math.PI + (i * stepSize);

    const launchPair = (t: number) => {
       const x = 16 * Math.pow(Math.sin(t), 3);
       const y = -(13 * Math.cos(t) - 5 * Math.cos(2 * t) - 2 * Math.cos(3 * t) - Math.cos(4 * t));
       const targetX = centerX + x * scale;
       const targetY = centerY + y * scale;
       const startX = width / 2 + (Math.random() - 0.5) * 200;
       fireworks.push(new Firework(startX, height, targetX, targetY, true));
    }

    setTimeout(() => {
      launchPair(t1);
      launchPair(t2);
    }, i * 100); 
  }

  for(let j = -2; j <= 2; j++) {
      const tx = width/2 + j * (width * 0.15);
      const ty = height * 0.6 + Math.abs(j) * 50;
      setTimeout(() => {
         fireworks.push(new Firework(width/2, height, tx, ty, true));
      }, 2200 + Math.abs(j) * 200);
  }
};

// 左侧打字机
const startTypewriter = () => {
  isFinalTextTyping.value = true;
  let index = 0;
  const interval = setInterval(() => {
    if (index < fullFinalText.length) {
      displayedFinalText.value += fullFinalText[index];
      index++;
    } else {
      clearInterval(interval);
      isFinalTextTyping.value = false;
    }
  }, TYPE_SPEED); 
};

// --- 烟花粒子系统 ---
const CONFIG = {
  particleCount: 300, gravity: 0.06, friction: 0.96, skyColor: '#050510', horizonColor: '#1a1a3a',
};

class Particle {
  x: number; y: number; vx: number; vy: number; alpha: number; color: string; decay: number; size: number;
  constructor(x: number, y: number, hue: number, isSpecial: boolean) {
    this.x = x; this.y = y;
    const angle = Math.random() * Math.PI * 2;
    const speed = Math.random() * 5 + 2; 
    this.vx = Math.cos(angle) * speed; this.vy = Math.sin(angle) * speed;
    this.alpha = 1; 
    if (isSpecial) {
      this.decay = Math.random() * 0.005 + 0.001; 
      this.size = Math.random() * 1 + 0.5;        
      const brightness = Math.random() * 20 + 60; 
      this.color = `hsl(${hue}, 100%, ${brightness}%)`;
    } else {
      this.decay = Math.random() * 0.015 + 0.005; 
      this.size = Math.random() * 2 + 1;          
      const brightness = Math.random() * 20 + 50;
      this.color = `hsl(${hue}, 100%, ${brightness}%)`;
    }
  }
  update() {
    this.vx *= CONFIG.friction; this.vy *= CONFIG.friction; this.vy += CONFIG.gravity;
    this.x += this.vx; this.y += this.vy; this.alpha -= this.decay;
  }
  draw(ctx: CanvasRenderingContext2D) {
    ctx.globalAlpha = this.alpha; ctx.fillStyle = this.color;
    ctx.beginPath(); ctx.arc(this.x, this.y, this.size, 0, Math.PI * 2); ctx.fill();
  }
}

class Firework {
  x: number; y: number; sx: number; sy: number; tx: number; ty: number;
  distanceToTarget: number; distanceTraveled: number = 0;
  coordinates: {x: number, y: number}[] = [];
  angle: number; speed: number = 2; acceleration: number = 1.05;
  brightness: number; targetRadius: number = 1; hue: number; exploded: boolean = false;
  isSpecial: boolean;

  constructor(sx: number, sy: number, tx: number, ty: number, isSpecial: boolean = false) {
    this.x = sx; this.y = sy; this.sx = sx; this.sy = sy; this.tx = tx; this.ty = ty;
    this.isSpecial = isSpecial;
    this.distanceToTarget = Math.sqrt(Math.pow(tx - sx, 2) + Math.pow(ty - sy, 2));
    this.angle = Math.atan2(ty - sy, tx - sx);
    this.brightness = Math.random() * 50 + 50;
    
    if (this.isSpecial) {
       this.hue = 330 + Math.random() * 20; 
    } else {
       this.hue = Math.floor(Math.random() * 360);
    }
    while(this.coordinates.length < 3) { this.coordinates.push({x: sx, y: sy}); }
  }

  update(index: number) {
    this.coordinates.pop(); this.coordinates.unshift({x: this.x, y: this.y});
    this.speed *= this.acceleration;
    const vx = Math.cos(this.angle) * this.speed;
    const vy = Math.sin(this.angle) * this.speed;
    this.distanceTraveled = Math.sqrt(Math.pow(this.sx - (this.x + vx), 2) + Math.pow(this.sy - (this.y + vy), 2));
    if (this.distanceTraveled >= this.distanceToTarget) {
      createParticles(this.tx, this.ty, this.hue, this.isSpecial);
      fireworks.splice(index, 1);
    } else { this.x += vx; this.y += vy; }
  }
  draw(ctx: CanvasRenderingContext2D) {
    ctx.beginPath();
    ctx.moveTo(this.coordinates[this.coordinates.length - 1].x, this.coordinates[this.coordinates.length - 1].y);
    ctx.lineTo(this.x, this.y);
    ctx.strokeStyle = `hsl(${this.hue}, 100%, ${this.brightness}%)`;
    ctx.stroke();
  }
}

let fireworks: Firework[] = [];
let particles: Particle[] = [];
let stars: {x: number, y: number, size: number, alpha: number, blink: number}[] = [];

function createParticles(x: number, y: number, hue: number, isSpecial: boolean) {
  for (let i = 0; i < CONFIG.particleCount; i++) { 
    particles.push(new Particle(x, y, hue, isSpecial)); 
  }
}

function drawBackground() {
  if (!ctx) return;
  const gradient = ctx.createLinearGradient(0, 0, 0, height);
  gradient.addColorStop(0, CONFIG.skyColor);
  gradient.addColorStop(0.85, CONFIG.horizonColor); 
  gradient.addColorStop(1, '#050510'); 
  ctx.fillStyle = gradient; ctx.fillRect(0, 0, width, height);
  ctx.fillStyle = '#FFF';
  stars.forEach(star => {
    ctx!.globalAlpha = 0.5 + Math.sin(Date.now() * 0.005 + star.blink) * 0.5; 
    ctx!.beginPath(); ctx!.arc(star.x, star.y, star.size, 0, Math.PI * 2); ctx!.fill();
  });
  ctx.globalAlpha = 1;
}

function drawCouple() {
  if (!ctx) return;
  const centerX = width / 2; const bottomY = height;
  const scale = Math.min(width, height) * 0.0008;
  ctx.save(); ctx.translate(centerX, bottomY); ctx.scale(scale, scale);
  const gradient = ctx.createRadialGradient(0, -450, 30, 0, -300, 600);
  gradient.addColorStop(0, "rgba(30, 30, 50, 0.95)");
  gradient.addColorStop(0.7, "rgba(10, 10, 20, 0.9)");
  gradient.addColorStop(1, "rgba(5, 5, 10, 0.9)");
  ctx.fillStyle = gradient;
  ctx.strokeStyle = "rgba(255, 230, 200, 0.5)";
  ctx.lineWidth = 6; ctx.lineJoin = "round";

  const boy = new Path2D();
  boy.moveTo(-130, 0); boy.quadraticCurveTo(-120, -350, -50, -350); 
  boy.lineTo(10, -350); boy.quadraticCurveTo(60, -350, 80, 0);       
  boy.closePath(); boy.moveTo(-20 + 45, -420); boy.arc(-20, -420, 55, 0, Math.PI * 2);
  ctx.fill(boy); ctx.stroke(boy);

  const girl = new Path2D();
  girl.moveTo(60, 0); girl.quadraticCurveTo(50, -300, 80, -320);  
  girl.lineTo(120, -320); girl.quadraticCurveTo(160, -280, 150, 0);   
  girl.closePath();
  const gx = 90; const gy = -390; const gr = 50;
  girl.moveTo(gx + gr, gy); girl.arc(gx, gy, gr, 0, Math.PI * 2);
  girl.moveTo(gx + 35 + 20, gy - 35); girl.arc(gx + 35, gy - 35, 20, 0, Math.PI * 2);
  ctx.fill(girl); ctx.stroke(girl);

  ctx.fillStyle = "rgba(255, 200, 200, 0.8)";
  const heart = new Path2D();
  const hx = 35, hy = -500;
  heart.moveTo(hx, hy);
  heart.bezierCurveTo(hx - 20, hy - 20, hx - 20, hy + 10, hx, hy + 25);
  heart.bezierCurveTo(hx + 20, hy + 10, hx + 20, hy - 20, hx, hy);
  ctx.fill(heart);
  ctx.restore();
}

function drawReflection() {
  if (!ctx) return;
  const horizonY = height * 0.85; 
  ctx.fillStyle = '#010105'; ctx.fillRect(0, horizonY, width, height - horizonY);
  for (let i = 0; i < particles.length; i++) {
    const p = particles[i];
    if (p.y < horizonY) {
      const distanceToHorizon = horizonY - p.y;
      const reflectionY = horizonY + distanceToHorizon; 
      const waveX = Math.sin(Date.now() * 0.005 + p.y * 0.05) * 3;
      ctx.globalAlpha = p.alpha * 0.05; ctx.fillStyle = p.color;
      ctx.beginPath(); ctx.ellipse(p.x + waveX, reflectionY, p.size * 2, p.size * 0.3, 0, 0, Math.PI * 2); ctx.fill();
    }
  }
  ctx.globalAlpha = 1;
}

function loop() {
  if (!ctx) return;
  animationId = requestAnimationFrame(loop);
  ctx.globalCompositeOperation = 'destination-out';
  ctx.fillStyle = 'rgba(0, 0, 0, 0.2)'; ctx.fillRect(0, 0, width, height);
  ctx.globalCompositeOperation = 'source-over';
  drawBackground();

  if (hasStarted.value && isRandomFireworksActive.value) {
    if (Math.random() < 0.03) { 
      const startX = Math.random() * (width * 0.8) + width * 0.1;
      const targetX = Math.random() * (width * 0.6) + width * 0.2;
      const targetY = Math.random() * (height * 0.5); 
      fireworks.push(new Firework(startX, height, targetX, targetY));
    }
  }

  let i = fireworks.length; while(i--) { fireworks[i].draw(ctx); fireworks[i].update(i); }
  ctx.globalCompositeOperation = 'lighter'; drawReflection();
  let j = particles.length; while(j--) { particles[j].draw(ctx); particles[j].update(); if (particles[j].alpha <= 0) { particles.splice(j, 1); } }
  ctx.globalCompositeOperation = 'source-over'; drawCouple();
}

onMounted(() => {
  const canvas = canvasRef.value;
  if (!canvas) return;
  ctx = canvas.getContext('2d');
  if (!ctx) return;
  const initSize = () => { width = window.innerWidth; height = window.innerHeight; canvas.width = width; canvas.height = height; };
  initSize();
  window.addEventListener('resize', initSize);
  for(let i=0; i<200; i++) {
    stars.push({ x: Math.random() * width, y: Math.random() * height * 0.8, size: Math.random() * 1.5, alpha: Math.random(), blink: Math.random() * Math.PI });
  }
  loop();
});

onUnmounted(() => { cancelAnimationFrame(animationId); });
</script>

<style scoped>
/* ... (引入字体部分保持不变) ... */
@import url('https://fonts.googleapis.com/css2?family=Dancing+Script:wght@400;700&display=swap');
@import url('https://fonts.googleapis.com/css2?family=Ma+Shan+Zheng&display=swap');

.cinema-container { position: fixed; top: 0; left: 0; width: 100vw; height: 100vh; background: #000; overflow: hidden; z-index: 9999; cursor: pointer; }
canvas { display: block; }

/* Intro */
.intro-overlay { position: absolute; top: 0; left: 0; width: 100%; height: 100%; display: flex; justify-content: center; align-items: center; z-index: 10002; pointer-events: none; }
.intro-text-wrapper { text-align: center; color: rgba(255, 255, 255, 0.9); animation: fadeInPulse 3s ease-in-out infinite; }
.intro-text { font-family: "Songti SC", "SimSun", serif; font-size: 2rem; letter-spacing: 5px; margin-bottom: 20px; font-weight: bold; }

/* Main Content */
.overlay-content { position: absolute; top: 0; left: 0; width: 100%; height: 70%; display: flex; flex-direction: column; justify-content: center; align-items: center; pointer-events: none; z-index: 10000; }
.text-wrapper { text-align: center; color: white; } /* 去掉了之前的 animation fadeUpIn，改用内部元素控制 */

.title { 
  font-family: "Times New Roman", serif; 
  font-size: clamp(2rem, 5vw, 4rem); 
  font-weight: normal; 
  margin-bottom: 2rem; /* 增加下边距，给副标题留足空间 */
  text-shadow: 0 0 20px rgba(255, 215, 0, 0.5); 
  letter-spacing: 2px; 
  opacity: 1; 
}

/* --- ⭐ 修复后的打字机 CSS --- */
.handwritten-title { 
  font-family: 'Dancing Script', cursive; 
  overflow: hidden; 
  white-space: nowrap; 
  border-right: 0.15em solid orange; 
  width: 0; /* 默认宽度为0，不可见 */
  text-shadow: 0 0 25px rgba(255, 240, 180, 0.7), 0 0 5px rgba(255, 240, 180, 0.5); 
  letter-spacing: normal; 
  margin: 0 auto 2rem auto; /* 保持下边距 */
}

/* 只有加上这个类名时，才开始播放动画 */
.handwritten-title.start-typing {
  animation: 
    typing 4s steps(30, end) forwards, /* forwards 保持结束状态 */
    blink-caret 0.75s step-end 8 forwards;
}

/* --- ⭐ 副标题容器：占位但透明 --- */
.subtitle-container {
  opacity: 0; /* 默认透明 */
  transform: translateY(20px);
  transition: all 2s ease; /* 渐显过渡 */
}

.subtitle-container.visible {
  opacity: 1; /* 显示 */
  transform: translateY(0);
}

.subtitle { font-family: "PingFang SC", "Microsoft YaHei", sans-serif; font-size: clamp(1rem, 2vw, 1.5rem); font-weight: 300; color: rgba(255, 255, 255, 0.8); letter-spacing: 4px; margin-bottom: 1.5rem; }
.date { font-size: 0.9rem; color: rgba(255, 255, 255, 0.5); letter-spacing: 2px; font-family: monospace; margin-bottom: 1rem; }

/* Bottom Text */
.bottom-text-overlay { position: absolute; bottom: 5vh; left: 0; width: 100%; text-align: center; z-index: 10001; opacity: 0; animation: fadeUpIn 2s ease-out 1.5s forwards; }
.new-chinese-text { font-family: "Noto Serif SC", "Songti SC", "SimSun", "Times New Roman", serif; font-size: clamp(1.2rem, 3vw, 2rem); color: rgba(255, 255, 255, 0.9); font-weight: 600; letter-spacing: 10px; text-shadow: 0 2px 4px rgba(0, 0, 0, 0.8); margin: 0; }

/* Left Text */
.final-text-left { position: absolute; top: 50%; left: 5%; transform: translateY(-50%); max-width: 300px; text-align: left; z-index: 10005; pointer-events: none; }
.final-text-left p { font-family: "Songti SC", "SimSun", serif; font-size: 1.3rem; line-height: 1.8; color: rgba(255, 255, 255, 0.85); text-shadow: 0 2px 4px rgba(0,0,0,0.8); }
.cursor { display: inline-block; width: 2px; height: 1em; background-color: white; vertical-align: text-bottom; animation: blink-caret 0.75s step-end infinite; }

/* Animations */
@keyframes fadeUpIn { from { opacity: 0; transform: translateY(30px); } to { opacity: 1; transform: translateY(0); } }
@keyframes typing { from { width: 0 } to { width: 100% } }
@keyframes blink-caret { from, to { border-color: transparent; } 50% { border-color: orange; } }
@keyframes fadeInPulse { 0% { opacity: 0; transform: scale(0.95); } 50% { opacity: 1; transform: scale(1); } 100% { opacity: 0; transform: scale(0.95); } }

.fade-enter-active, .fade-leave-active { transition: opacity 0.8s ease; }
.fade-enter-from, .fade-leave-to { opacity: 0; }
.fade-slow-enter-active { transition: opacity 2s ease; }
.fade-slow-enter-from { opacity: 0; }
</style>