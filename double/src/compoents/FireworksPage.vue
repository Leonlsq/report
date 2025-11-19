<template>
  <div class="cinema-container">
    <canvas ref="canvasRef"></canvas>
    
    <div class="overlay-content">
      <div class="text-wrapper">
        <h1 class="title handwritten-title">Happy Birthday, My Love</h1>
        <p class="subtitle">愿我们的爱，如这漫天烟火，绚烂且长明。</p>
        <p class="date">2025.11.28</p> 
        </div>
    </div>

    <div class="bottom-text-overlay">
      <p class="new-chinese-text">相遇已是上签，再相遇便是一辈子。</p>
    </div>
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

// --- 配置参数 ---
const CONFIG = {
  particleCount: 300, 
  gravity: 0.06,      
  friction: 0.96,     
  skyColor: '#050510', 
  horizonColor: '#1a1a3a',
};

// --- 粒子系统 ---
class Particle {
  x: number;
  y: number;
  vx: number;
  vy: number;
  alpha: number;
  color: string;
  decay: number;
  size: number;

  constructor(x: number, y: number, hue: number) {
    this.x = x;
    this.y = y;
    const angle = Math.random() * Math.PI * 2;
    const speed = Math.random() * 5 + 2; 
    this.vx = Math.cos(angle) * speed;
    this.vy = Math.sin(angle) * speed;
    this.alpha = 1;
    this.decay = Math.random() * 0.015 + 0.005; 
    const brightness = Math.random() * 20 + 50;
    this.color = `hsl(${hue}, 100%, ${brightness}%)`;
    this.size = Math.random() * 2 + 1;
  }

  update() {
    this.vx *= CONFIG.friction;
    this.vy *= CONFIG.friction;
    this.vy += CONFIG.gravity;
    this.x += this.vx;
    this.y += this.vy;
    this.alpha -= this.decay;
  }

  draw(ctx: CanvasRenderingContext2D) {
    ctx.globalAlpha = this.alpha;
    ctx.fillStyle = this.color;
    ctx.beginPath();
    ctx.arc(this.x, this.y, this.size, 0, Math.PI * 2);
    ctx.fill();
  }
}

class Firework {
  x: number;
  y: number;
  sx: number;
  sy: number;
  tx: number;
  ty: number;
  distanceToTarget: number;
  distanceTraveled: number = 0;
  coordinates: {x: number, y: number}[] = [];
  angle: number;
  speed: number = 2;
  acceleration: number = 1.05;
  brightness: number;
  targetRadius: number = 1;
  hue: number;
  exploded: boolean = false;

  constructor(sx: number, sy: number, tx: number, ty: number) {
    this.x = sx;
    this.y = sy;
    this.sx = sx;
    this.sy = sy;
    this.tx = tx;
    this.ty = ty;
    this.distanceToTarget = Math.sqrt(Math.pow(tx - sx, 2) + Math.pow(ty - sy, 2));
    this.angle = Math.atan2(ty - sy, tx - sx);
    this.brightness = Math.random() * 50 + 50;
    this.hue = Math.floor(Math.random() * 360);
    while(this.coordinates.length < 3) {
      this.coordinates.push({x: sx, y: sy});
    }
  }

  update(index: number) {
    this.coordinates.pop();
    this.coordinates.unshift({x: this.x, y: this.y});
    this.speed *= this.acceleration;
    const vx = Math.cos(this.angle) * this.speed;
    const vy = Math.sin(this.angle) * this.speed;
    this.distanceTraveled = Math.sqrt(Math.pow(this.sx - (this.x + vx), 2) + Math.pow(this.sy - (this.y + vy), 2));

    if (this.distanceTraveled >= this.distanceToTarget) {
      createParticles(this.tx, this.ty, this.hue);
      fireworks.splice(index, 1);
    } else {
      this.x += vx;
      this.y += vy;
    }
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

function createParticles(x: number, y: number, hue: number) {
  for (let i = 0; i < CONFIG.particleCount; i++) {
    particles.push(new Particle(x, y, hue));
  }
}

// --- 绘制场景 ---

function drawBackground() {
  if (!ctx) return;
  const gradient = ctx.createLinearGradient(0, 0, 0, height);
  gradient.addColorStop(0, CONFIG.skyColor);
  gradient.addColorStop(0.8, CONFIG.horizonColor); 
  gradient.addColorStop(1, '#0a0a1a'); 
  
  ctx.fillStyle = gradient;
  ctx.fillRect(0, 0, width, height);

  ctx.fillStyle = '#FFF';
  stars.forEach(star => {
    ctx!.globalAlpha = 0.5 + Math.sin(Date.now() * 0.005 + star.blink) * 0.5; 
    ctx!.beginPath();
    ctx!.arc(star.x, star.y, star.size, 0, Math.PI * 2);
    ctx!.fill();
  });
  ctx.globalAlpha = 1;
}

// ⭐⭐⭐ 核心修改：重塑人物结构 ⭐⭐⭐
function drawCouple() {
  if (!ctx) return;

  const centerX = width / 2;
  const bottomY = height;
  
  // ⭐⭐⭐ 修改点：放大比例 ⭐⭐⭐
  // 从 0.00042 改为 0.0008，让人物占据画面约 1/3 到 1/2 的高度
  const scale = Math.min(width, height) * 0.0008;

  ctx.save();
  ctx.translate(centerX, bottomY);
  ctx.scale(scale, scale);

  // --- 依然使用那个唯美的渐变 ---
  const gradient = ctx.createRadialGradient(
    0, -450, 30,
    0, -300, 600
  );
  gradient.addColorStop(0, "rgba(30, 30, 50, 0.95)");
  gradient.addColorStop(0.7, "rgba(10, 10, 20, 0.9)");
  gradient.addColorStop(1, "rgba(5, 5, 10, 0.9)");

  ctx.fillStyle = gradient;
  
  // 高光描边设置
  ctx.strokeStyle = "rgba(255, 230, 200, 0.5)";
  ctx.lineWidth = 6;
  ctx.lineJoin = "round";

  // ==========================================
  // 1. 绘制男生 (左边，稍微大一点)
  // ==========================================
  const boy = new Path2D();
  
  // 男生身体：一个稳重的"面包"形状
  boy.moveTo(-130, 0); 
  boy.quadraticCurveTo(-120, -350, -50, -350); // 左侧身体线
  boy.lineTo(10, -350);                        // 肩膀是平的
  boy.quadraticCurveTo(60, -350, 80, 0);       // 右侧身体线
  boy.closePath();

  // 男生头：正圆
  boy.moveTo(-20 + 45, -420); // 移动笔触避免连线
  boy.arc(-20, -420, 55, 0, Math.PI * 2);

  // 渲染男生
  ctx.fill(boy);
  ctx.stroke(boy);


  // ==========================================
  // 2. 绘制女生 (右边，稍微小一点，靠着男生)
  // ==========================================
  const girl = new Path2D();

  // 女生身体：稍微瘦一点的"面包"
  girl.moveTo(60, 0);
  // 注意：这里的控制点让身体向左倾斜一点，制造"依偎"感
  girl.quadraticCurveTo(50, -300, 80, -320);  // 左肩位置
  girl.lineTo(120, -320);                     // 肩膀
  girl.quadraticCurveTo(160, -280, 150, 0);   // 右侧
  girl.closePath();

  // 女生头：正圆 (位置靠左，挨着男生的头)
  const gx = 90;
  const gy = -390;
  const gr = 50;
  girl.moveTo(gx + gr, gy);
  girl.arc(gx, gy, gr, 0, Math.PI * 2);

  // 女生发型特征：丸子头 (Bun) 
  girl.moveTo(gx + 35 + 20, gy - 35); 
  girl.arc(gx + 35, gy - 35, 20, 0, Math.PI * 2);

  // 渲染女生
  // 这里的 fill 会覆盖掉男生重叠的部分，看起来更有层次
  ctx.fill(girl);
  ctx.stroke(girl);

  // ==========================================
  // 3. 增加一点"卡通感"的小细节
  // ==========================================
  
  // 在两个头中间画一个小爱心
  ctx.fillStyle = "rgba(255, 200, 200, 0.8)";
  const heart = new Path2D();
  const hx = 35, hy = -500;
  // 简易爱心路径
  heart.moveTo(hx, hy);
  heart.bezierCurveTo(hx - 20, hy - 20, hx - 20, hy + 10, hx, hy + 25);
  heart.bezierCurveTo(hx + 20, hy + 10, hx + 20, hy - 20, hx, hy);
  ctx.fill(heart);

  ctx.restore();
}

// 辅助函数：绘制星星 (Q版细节常用的几何图形)
function drawStar(ctx, cx, cy, spikes, outerRadius, innerRadius) {
  let rot = Math.PI / 2 * 3;
  let x = cx;
  let y = cy;
  let step = Math.PI / spikes;

  ctx.beginPath();
  ctx.moveTo(cx, cy - outerRadius);
  for (let i = 0; i < spikes; i++) {
    x = cx + Math.cos(rot) * outerRadius;
    y = cy + Math.sin(rot) * outerRadius;
    ctx.lineTo(x, y);
    rot += step;

    x = cx + Math.cos(rot) * innerRadius;
    y = cy + Math.sin(rot) * innerRadius;
    ctx.lineTo(x, y);
    rot += step;
  }
  ctx.closePath();
  ctx.fill();
}

function drawReflection() {
  if (!ctx) return;
  const horizonY = height * 0.85; 

  ctx.fillStyle = '#03030a';
  ctx.fillRect(0, horizonY, width, height - horizonY);

  for (let i = 0; i < particles.length; i++) {
    const p = particles[i];
    if (p.y < horizonY) {
      const distanceToHorizon = horizonY - p.y;
      const reflectionY = horizonY + distanceToHorizon; 
      const waveX = Math.sin(Date.now() * 0.005 + p.y * 0.05) * 10;
      
      ctx.globalAlpha = p.alpha * 0.25; 
      ctx.fillStyle = p.color;
      ctx.beginPath();
      ctx.ellipse(p.x + waveX, reflectionY, p.size * 2, p.size * 0.5, 0, 0, Math.PI * 2);
      ctx.fill();
    }
  }
  ctx.globalAlpha = 1;
}

function loop() {
  if (!ctx) return;
  
  animationId = requestAnimationFrame(loop);

  ctx.globalCompositeOperation = 'destination-out';
  ctx.fillStyle = 'rgba(0, 0, 0, 0.2)'; 
  ctx.fillRect(0, 0, width, height);
  
  ctx.globalCompositeOperation = 'source-over';
  drawBackground();

  if (Math.random() < 0.03) { 
    const startX = Math.random() * (width * 0.8) + width * 0.1;
    const targetX = Math.random() * (width * 0.6) + width * 0.2;
    const targetY = Math.random() * (height * 0.5); 
    fireworks.push(new Firework(startX, height, targetX, targetY));
  }

  let i = fireworks.length;
  while(i--) {
    fireworks[i].draw(ctx);
    fireworks[i].update(i);
  }

  ctx.globalCompositeOperation = 'lighter'; 
  drawReflection();

  let j = particles.length;
  while(j--) {
    particles[j].draw(ctx);
    particles[j].update();
    if (particles[j].alpha <= 0) {
      particles.splice(j, 1);
    }
  }

  ctx.globalCompositeOperation = 'source-over'; 
  drawCouple();
}

onMounted(() => {
  const canvas = canvasRef.value;
  if (!canvas) return;
  
  ctx = canvas.getContext('2d');
  if (!ctx) return;

  const initSize = () => {
    width = window.innerWidth;
    height = window.innerHeight;
    canvas.width = width;
    canvas.height = height;
  };
  initSize();
  window.addEventListener('resize', initSize);

  for(let i=0; i<200; i++) {
    stars.push({
      x: Math.random() * width,
      y: Math.random() * height * 0.8, 
      size: Math.random() * 1.5,
      alpha: Math.random(),
      blink: Math.random() * Math.PI
    });
  }

  loop();
});

onUnmounted(() => {
  cancelAnimationFrame(animationId);
});
</script>

<style scoped>
/* ⚠️ @import 必须放在 CSS 的第一行才能生效 */
@import url('https://fonts.googleapis.com/css2?family=Dancing+Script:wght@400;700&display=swap');
@import url('https://fonts.googleapis.com/css2?family=Ma+Shan+Zheng&display=swap');

.cinema-container {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: #000;
  overflow: hidden;
  z-index: 9999;
}

canvas {
  display: block;
}

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

.text-wrapper {
  text-align: center;
  color: white;
  opacity: 0;
  animation: fadeUpIn 3s ease-out 2s forwards; 
}

.title {
  font-family: "Times New Roman", serif; 
  font-size: clamp(2rem, 5vw, 4rem);
  font-weight: normal;
  margin-bottom: 1rem;
  text-shadow: 0 0 20px rgba(255, 215, 0, 0.5); 
  letter-spacing: 2px;
  opacity: 1; 
}

/* --- ⭐ 修改点：光标动画 ⭐ --- */
.handwritten-title {
  font-family: 'Dancing Script', cursive; 
  overflow: hidden; 
  white-space: nowrap; 
  border-right: 0.15em solid orange; 
  animation: 
    typing 4s steps(22, end) 5s forwards, 
    /* 修改这里：由 infinite 改为 13 forwards */
    /* 逻辑：闪烁 13 次 (约9.75秒) 后停止，forwards 保持最后一帧(透明)，从而实现消失 */
    blink-caret 0.75s step-end 13 forwards;
  width: 0; 
  text-shadow: 0 0 25px rgba(255, 240, 180, 0.7), 0 0 5px rgba(255, 240, 180, 0.5); 
  letter-spacing: normal; 
}

.subtitle {
  font-family: "PingFang SC", "Microsoft YaHei", sans-serif;
  font-size: clamp(1rem, 2vw, 1.5rem);
  font-weight: 300;
  color: rgba(255, 255, 255, 0.8);
  letter-spacing: 4px;
  margin-bottom: 1.5rem;
}

.date {
  font-size: 0.9rem;
  color: rgba(255, 255, 255, 0.5);
  letter-spacing: 2px;
  font-family: monospace;
  margin-bottom: 1rem; 
}

/* 底部文字容器 */
.bottom-text-overlay {
  position: absolute;
  bottom: 5vh; 
  left: 0;
  width: 100%;
  text-align: center;
  z-index: 10001;
  opacity: 0;
  animation: fadeUpIn 3s ease-out 3s forwards; 
}

.new-chinese-text {
  /* ⭐ 优先使用衬线体：思源宋体 / 标准宋体 ⭐ */
  font-family: "Noto Serif SC", "Songti SC", "SimSun", "Times New Roman", serif;
  
  font-size: clamp(1.2rem, 3vw, 2rem); 
  color: rgba(255, 255, 255, 0.9);
  font-weight: 600; /* 宋体横细竖粗，稍微加粗一点在黑色背景上更清晰 */
  letter-spacing: 10px; /* 宋体拉大间距非常有电影感 */
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.8);
  margin: 0;
}

@keyframes fadeUpIn {
  from {
    opacity: 0;
    transform: translateY(30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes typing {
  from { width: 0 }
  to { width: 100% }
}

@keyframes blink-caret {
  from, to { border-color: transparent }
  50% { border-color: orange; }
}
</style>