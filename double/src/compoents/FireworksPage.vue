<template>
  <div class="cinema-container" @click="startShow">
    <canvas ref="canvasRef"></canvas>
    
    <transition name="fade">
      <div v-if="!hasStarted" class="intro-overlay">
        <div class="intro-text-wrapper">
          <p class="intro-text">轻触屏幕 开启生日惊喜</p>
        </div>
      </div>
    </transition>

    <div v-if="hasStarted" class="overlay-content">
      <div class="text-wrapper">
        <h1 class="title handwritten-title" :class="{ 'start-typing': showCenterTitle }">
          Happy Birthday, My Lovely Ty
        </h1>

        <div class="subtitle-container" :class="{ 'visible': showCenterSubtitle }">
          <p class="subtitle">
            现实的烟花易冷，但我为你写的这一场，永远不会熄灭。<br />
            愿我们的爱，如这漫天烟火，绚烂且长明。
          </p>
          <p class="date">2025.11.28</p> 
        </div>
      </div>
    </div>

    <transition name="fade-slow">
      <div v-if="showBottomText" class="bottom-text-overlay">
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

    <transition name="fade-slide-left">
      <div v-if="showRightClock" class="right-clock-container">
        <p class="clock-label">距离下一次见面</p>
        <p class="clock-date">(2026.02.26)</p>
        
        <div class="flip-clock">
          <div class="flip-unit">
            <div class="flip-card">
              <transition name="flip" mode="out-in">
                <span :key="timeLeft.days">{{ formatNum(timeLeft.days) }}</span>
              </transition>
            </div>
            <span class="unit-label">DAYS</span>
          </div>
          <div class="flip-unit">
            <div class="flip-card">
              <transition name="flip" mode="out-in">
                <span :key="timeLeft.hours">{{ formatNum(timeLeft.hours) }}</span>
              </transition>
            </div>
            <span class="unit-label">HRS</span>
          </div>
          <div class="flip-unit">
            <div class="flip-card">
              <transition name="flip" mode="out-in">
                <span :key="timeLeft.minutes">{{ formatNum(timeLeft.minutes) }}</span>
              </transition>
            </div>
            <span class="unit-label">MIN</span>
          </div>
          <div class="flip-unit seconds-unit">
            <div class="flip-card">
              <transition name="flip" mode="out-in">
                <span :key="timeLeft.seconds">{{ formatNum(timeLeft.seconds) }}</span>
              </transition>
            </div>
            <span class="unit-label">SEC</span>
          </div>
        </div>
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
const showRightClock = ref(false); // 右侧时钟

// 左侧打字机相关
const displayedFinalText = ref(''); 
// \u3000 代表一个汉字宽度的空格
const fullFinalText = "\u3000\u3000其实我不记得和AI一起改了多少个 Bug， 也不记得熬了几个夜。 我只记得，在敲下每一行代码的时候， 脑海里全是你看到它时惊喜的样子。 只要你笑了，这一切就有了意义。\n\u3000\u3000汤悦，生日快乐，往后余生，年年岁岁有我。";
const isFinalTextTyping = ref(false);
const TYPE_SPEED = 150; // 稍微调快一点打字速度，让iPad用户等待时间变短

// 倒计时相关
const timeLeft = ref({ days: 0, hours: 0, minutes: 0, seconds: 0 });
let timer: any = null;
const targetDate = new Date('2026-02-26T00:00:00');

const updateCountdown = () => {
  const current = new Date();
  let diff = targetDate.getTime() - current.getTime();
  if (diff < 0) diff = 0;
  timeLeft.value.days = Math.floor(diff / (1000 * 60 * 60 * 24));
  timeLeft.value.hours = Math.floor((diff % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60));
  timeLeft.value.minutes = Math.floor((diff % (1000 * 60 * 60)) / (1000 * 60));
  timeLeft.value.seconds = Math.floor((diff % (1000 * 60)) / 1000);
};
const formatNum = (num: number) => num.toString().padStart(2, '0');

// --- ⭐ 终极时间轴控制 ⭐ ---
const startShow = () => {
  if (!hasStarted.value) {
    hasStarted.value = true;
    isRandomFireworksActive.value = true; 
    
    // 启动倒计时计时器
    updateCountdown();
    timer = setInterval(updateCountdown, 1000);

    // 1. T+0s: 显示主标题
    showCenterTitle.value = true;

    // 2. T+3s: 左侧独白开始打字 (稍微提前一点)
    const delayLeftText = 3000; 
    setTimeout(() => {
      showFinalTextContainer.value = true;
      startTypewriter();
    }, delayLeftText);

    // 动态计算打字时长
    const typingDuration = fullFinalText.length * TYPE_SPEED; 
    const readBuffer = 1000; 
    
    // 3. T+Subtitle: 中间副标题
    const delaySubtitle = delayLeftText + typingDuration + readBuffer;
    setTimeout(() => {
      showCenterSubtitle.value = true;
    }, delaySubtitle);

    // 4. T+Bottom: 底部文字
    const delayBottom = delaySubtitle + 2000;
    setTimeout(() => {
      showBottomText.value = true;
    }, delayBottom);

    // 5. T+Clear: 暂停随机烟花
    const delayClear = delayBottom + 3000;
    setTimeout(() => {
      isRandomFireworksActive.value = false;
    }, delayClear);

    // 6. T+Heart: 发射爱心烟花
    const delayHeart = delayClear + 1000;
    setTimeout(() => {
      launchHeartPattern();
    }, delayHeart);

    // 7. T+Resume: 恢复随机烟花
    const heartDuration = 5000;
    setTimeout(() => {
      isRandomFireworksActive.value = true;
    }, delayHeart + heartDuration);

    // 8. T+Clock: 右侧时钟浮现
    const delayClock = delayHeart + heartDuration + 3000;
    setTimeout(() => {
      showRightClock.value = true;
    }, delayClock);
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

onUnmounted(() => { 
  cancelAnimationFrame(animationId); 
  if(timer) clearInterval(timer);
});
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Dancing+Script:wght@400;700&display=swap');
@import url('https://fonts.googleapis.com/css2?family=Ma+Shan+Zheng&display=swap');
@import url('https://fonts.googleapis.com/css2?family=Fira+Code:wght@500&display=swap');

.cinema-container { position: fixed; top: 0; left: 0; width: 100vw; height: 100vh; background: #000; overflow: hidden; z-index: 9999; cursor: pointer; }
canvas { display: block; }

/* Intro */
.intro-overlay { position: absolute; top: 0; left: 0; width: 100%; height: 100%; display: flex; justify-content: center; align-items: center; z-index: 10002; pointer-events: none; }
.intro-text-wrapper { text-align: center; color: rgba(255, 255, 255, 0.9); animation: fadeInPulse 3s ease-in-out infinite; }
.intro-text { font-family: "Songti SC", "SimSun", serif; font-size: 2rem; letter-spacing: 5px; margin-bottom: 20px; font-weight: bold; }

/* Main Content - Center Title */
.overlay-content { 
  position: absolute; 
  top: 0; 
  left: 0; 
  width: 100%; 
  height: 70%; 
  display: flex; 
  flex-direction: column; 
  justify-content: center; 
  align-items: center; 
  pointer-events: none; 
  z-index: 10000; 
}
.text-wrapper { text-align: center; color: white; }

.title { 
  font-family: "Times New Roman", serif; 
  font-size: clamp(2rem, 5vw, 4rem); 
  font-weight: normal; 
  margin-bottom: 2rem; 
  text-shadow: 0 0 20px rgba(255, 215, 0, 0.5); 
  letter-spacing: 2px; 
  opacity: 1; 
}

.handwritten-title { 
  font-family: 'Dancing Script', cursive; 
  overflow: hidden; 
  white-space: nowrap; 
  border-right: 0.15em solid orange; 
  width: 0; 
  text-shadow: 0 0 25px rgba(255, 240, 180, 0.7), 0 0 5px rgba(255, 240, 180, 0.5); 
  letter-spacing: normal; 
  margin: 0 auto 2rem auto; 
}

.handwritten-title.start-typing {
  animation: 
    typing 4s steps(30, end) forwards, 
    blink-caret 0.75s step-end 8 forwards;
}

/* Subtitle */
.subtitle-container {
  opacity: 0; 
  transform: translateY(20px);
  transition: all 2s ease; 
}
.subtitle-container.visible {
  opacity: 1; 
  transform: translateY(0);
}

.subtitle { 
  font-family: "PingFang SC", "Microsoft YaHei", sans-serif; 
  font-size: clamp(1rem, 2vw, 1.5rem); 
  font-weight: 300; 
  color: rgba(255, 255, 255, 0.8); 
  letter-spacing: 4px; 
  margin-bottom: 1.5rem;
  line-height: 1.6; 
}
.date { font-size: 0.9rem; color: rgba(255, 255, 255, 0.5); letter-spacing: 2px; font-family: monospace; margin-bottom: 1rem; }

/* Bottom Text */
.bottom-text-overlay { position: absolute; bottom: 5vh; left: 0; width: 100%; text-align: center; z-index: 10001; opacity: 0; animation: fadeUpIn 2s ease-out 1.5s forwards; }

/* Left Text (Deskop Default) */
.final-text-left { 
  position: absolute; 
  top: 50%; 
  left: 5%; 
  transform: translateY(-50%); 
  max-width: 300px; 
  text-align: left; 
  z-index: 10005; 
  pointer-events: none; 
}
.final-text-left p { 
  font-family: "Songti SC", "SimSun", serif; 
  font-size: 1.3rem; 
  line-height: 1.8; 
  color: rgba(255, 255, 255, 0.85); 
  text-shadow: 0 2px 4px rgba(0,0,0,0.8);
  white-space: pre-wrap; 
}
.cursor { display: inline-block; width: 2px; height: 1em; background-color: white; vertical-align: text-bottom; animation: blink-caret 0.75s step-end infinite; }

/* Right Clock (Desktop Default) */
.right-clock-container {
  position: absolute;
  top: 50%;
  right: 5%; 
  transform: translateY(-50%);
  z-index: 10005;
  pointer-events: none;
  text-align: center;
  
  background: rgba(20, 20, 40, 0.4);
  backdrop-filter: blur(8px);
  padding: 25px 30px;
  border-radius: 25px;
  border: 1px solid rgba(255, 255, 255, 0.1);
  box-shadow: 0 10px 30px rgba(0,0,0,0.3);
}
.clock-label { font-size: 1.1rem; color: rgba(255, 255, 255, 0.9); margin-bottom: 5px; letter-spacing: 2px; font-weight: 300; }
.clock-date { font-size: 0.9rem; color: rgba(255, 255, 255, 0.6); margin-bottom: 20px; letter-spacing: 1px; font-family: monospace; }
.flip-clock { display: flex; justify-content: center; gap: 15px; }
.flip-unit { display: flex; flex-direction: column; align-items: center; }
.flip-card {
  background: rgba(255, 255, 255, 0.15);
  border-radius: 12px; padding: 12px 8px; min-width: 50px;
  text-align: center; font-family: 'Fira Code', monospace; font-size: 28px; font-weight: 600; color: #fff;
  text-shadow: 0 2px 5px rgba(0,0,0,0.5); box-shadow: inset 0 0 10px rgba(255,255,255,0.05), 0 5px 15px rgba(0,0,0,0.2);
  position: relative; overflow: hidden; border-top: 1px solid rgba(255,255,255,0.2);
}
.seconds-unit .flip-card { background: rgba(255, 215, 0, 0.15); border: 1px solid rgba(255, 215, 0, 0.3); color: #ffedb3; }
.unit-label { font-size: 11px; margin-top: 8px; opacity: 0.7; font-weight: 500; letter-spacing: 1px; color: rgba(255,255,255,0.8); }

/* Animations */
@keyframes fadeUpIn { from { opacity: 0; transform: translateY(30px); } to { opacity: 1; transform: translateY(0); } }
@keyframes typing { from { width: 0 } to { width: 100% } }
@keyframes blink-caret { from, to { border-color: transparent; } 50% { border-color: orange; } }
@keyframes fadeInPulse { 0% { opacity: 0; transform: scale(0.95); } 50% { opacity: 1; transform: scale(1); } 100% { opacity: 0; transform: scale(0.95); } }
.fade-enter-active, .fade-leave-active { transition: opacity 0.8s ease; }
.fade-enter-from, .fade-leave-to { opacity: 0; }
.fade-slow-enter-active { transition: opacity 2s ease; }
.fade-slow-enter-from { opacity: 0; }
.fade-slide-left-enter-active { transition: all 1.5s ease-out; }
.fade-slide-left-enter-from { opacity: 0; transform: translate(30px, -50%); }
.flip-enter-active { animation: flipIn 0.6s cubic-bezier(0.25, 0.46, 0.45, 0.94); }
.flip-leave-active { display: none; }
@keyframes flipIn { from { opacity: 0; transform: perspective(400px) rotateX(-90deg); } to { opacity: 1; transform: perspective(400px) rotateX(0deg); } }

/* 🌟🌟🌟 核心修改区：平板/iPad 垂直布局适配 🌟🌟🌟 */
@media screen and (max-width: 1200px) {
  
  /* 1. 标题区：不再垂直居中，而是靠上 */
  .overlay-content {
    justify-content: flex-start;
    padding-top: 15vh; /* 距离顶部 15%，留出空间给烟花 */
    height: auto;
  }
  .title {
    font-size: 2.2rem; /* 字体适度缩小 */
    margin-bottom: 0.5rem;
  }
  .subtitle-container {
    margin-top: 10px;
  }
  .subtitle {
    font-size: 1rem;
    line-height: 1.6;
    max-width: 90%;
    margin: 0 auto;
  }

  /* 2. 左侧独白（现在变到中间）：垂直堆叠 */
  .final-text-left {
    position: absolute;
    top: 40%; /* 位于屏幕中间偏上 */
    left: 50%;
    transform: translate(-50%, 0); /* 水平居中 */
    width: 85%;
    max-width: 600px;
    
    /* 增加半透明黑色背景，防止文字与烟花重叠看不清 */
    background: rgba(0, 0, 0, 0.4); 
    backdrop-filter: blur(4px);
    padding: 20px;
    border-radius: 15px;
    text-align: center; /* iPad上居中对齐更有仪式感 */
    
    /* 确保层级高于烟花，但低于某些交互 */
    z-index: 10005;
  }
  
  .final-text-left p {
    font-size: 1.1rem;
    text-align: left; /* 保持左对齐阅读习惯，或者改成center看喜好 */
    display: inline-block; /* 配合父级text-align: center实现块居中 */
  }

  /* 3. 右侧倒计时（现在变到最下方）：垂直堆叠 */
  .right-clock-container {
    position: absolute;
    top: auto; /* 取消顶部定位 */
    bottom: 8vh; /* 固定在底部 */
    right: auto; 
    left: 50%;
    transform: translate(-50%, 0); /* 水平居中 */
    width: auto;
    
    /* 稍微缩小尺寸适配窄屏 */
    scale: 0.85;
    transform-origin: bottom center;
  }

  /* 4. 动画修正：从底部浮现而不是从右边滑入 */
  .fade-slide-left-enter-from { 
    opacity: 0; 
    transform: translate(-50%, 30px); /* 注意这里保持 translate(-50%) 的水平居中状态 */
  }
}

/* 📱 手机端适配 (如果需要更小的屏幕) */
@media screen and (max-width: 600px) {
  .title { font-size: 1.6rem; }
  .final-text-left { top: 35%; padding: 15px; width: 92%; }
  .final-text-left p { font-size: 0.95rem; }
  .right-clock-container { scale: 0.7; bottom: 5vh; }
}
</style>