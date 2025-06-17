<template>
  <div class="checker-container">
    <canvas ref="bgCanvas" class="bg-canvas"></canvas>
    <div class="main-content">
      <el-icon class="pc-icon" size="48" color="#409EFF">
        <Monitor />
      </el-icon>
      <h1 class="title">电脑开机状态检测器</h1>
      <p class="desc">使用先进的大数据技术分析您的电脑状态</p>
      <el-button
        class="detect-btn"
        type="primary"
        size="large"
        :loading="loading"
        @click="startDetect"
        round
      >
        <span v-if="!loading">开始检测</span>
        <span v-else>检测中...</span>
      </el-button>
      <transition name="fade">
        <div v-if="showResult" class="result-box">
          <el-icon color="#67C23A" size="32"><CircleCheckFilled /></el-icon>
          <span class="result-text">恭喜你， 你的电脑当前为开机状态！</span>
        </div>
      </transition>
      <footer class="footer">
        © 2025 电脑健康检测中心 | 使用先进的大数据技术
      </footer>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { Monitor, CircleCheckFilled } from '@element-plus/icons-vue'
import gsap from 'gsap'

const loading = ref(false)
const showResult = ref(false)
const bgCanvas = ref(null)

const startDetect = () => {
  loading.value = true
  showResult.value = false
  
  // 强化动效：按钮震动+发光
  gsap.to('.detect-btn', { scale: 1.1, yoyo: true, repeat: 5, duration: 0.1 })
  
  setTimeout(() => {
    loading.value = false
    showResult.value = true
    gsap.fromTo('.result-box', 
      { scale: 0.5, opacity: 0, rotationY: 180 }, 
      { scale: 1, opacity: 1, rotationY: 0, duration: 0.8, ease: 'back.out(1.7)' }
    )
    // 粒子爆发动画
    triggerParticles()
  }, 2200)
}

function triggerParticles() {
  const canvas = bgCanvas.value
  if (!canvas) return
  const ctx = canvas.getContext('2d')
  const particles = []
  const w = canvas.width
  const h = canvas.height
  
  // 增加粒子数量和多样性
  for (let i = 0; i < 120; i++) {
    particles.push({
      x: w / 2,
      y: h / 2,
      r: Math.random() * 8 + 2,
      color: `hsl(${120 + Math.random() * 120}, 85%, ${60 + Math.random() * 30}%)`,
      vx: (Math.random() - 0.5) * 15,
      vy: (Math.random() - 0.5) * 15,
      alpha: 1,
      gravity: Math.random() * 0.1,
      bounce: 0.7 + Math.random() * 0.3
    })
  }
  
  let frame = 0
  function animate() {
    ctx.clearRect(0, 0, w, h)
    
    for (const p of particles) {
      p.x += p.vx
      p.y += p.vy
      p.vy += p.gravity
      p.vx *= 0.98
      p.vy *= 0.98
      p.alpha *= 0.95
      
      // 边界反弹
      if (p.x < p.r || p.x > w - p.r) {
        p.vx *= -p.bounce
      }
      if (p.y < p.r || p.y > h - p.r) {
        p.vy *= -p.bounce
      }
      
      ctx.globalAlpha = p.alpha
      ctx.shadowBlur = 10
      ctx.shadowColor = p.color
      ctx.beginPath()
      ctx.arc(p.x, p.y, p.r, 0, Math.PI * 2)
      ctx.fillStyle = p.color
      ctx.fill()
      ctx.shadowBlur = 0
    }
    
    ctx.globalAlpha = 1
    frame++
    if (frame < 60) requestAnimationFrame(animate)
  }
  animate()
}

onMounted(() => {
  // 简化版3D背景
  const canvas = bgCanvas.value
  if (!canvas) return
  
  canvas.width = window.innerWidth
  canvas.height = window.innerHeight
  
  // 使用Canvas 2D代替Three.js避免兼容性问题
  const ctx = canvas.getContext('2d')
  const particles = []
  
  // 创建背景粒子
  for (let i = 0; i < 50; i++) {
    particles.push({
      x: Math.random() * canvas.width,
      y: Math.random() * canvas.height,
      vx: (Math.random() - 0.5) * 0.5,
      vy: (Math.random() - 0.5) * 0.5,
      size: Math.random() * 3 + 1,
      color: `hsl(${200 + Math.random() * 100}, 70%, 60%)`
    })
  }
  
  function animateBackground() {
    ctx.fillStyle = 'rgba(10, 10, 35, 0.1)'
    ctx.fillRect(0, 0, canvas.width, canvas.height)
    
    particles.forEach(p => {
      p.x += p.vx
      p.y += p.vy
      
      if (p.x < 0 || p.x > canvas.width) p.vx *= -1
      if (p.y < 0 || p.y > canvas.height) p.vy *= -1
      
      ctx.beginPath()
      ctx.arc(p.x, p.y, p.size, 0, Math.PI * 2)
      ctx.fillStyle = p.color
      ctx.fill()
    })
    
    requestAnimationFrame(animateBackground)
  }
  animateBackground()
  
  window.addEventListener('resize', () => {
    canvas.width = window.innerWidth
    canvas.height = window.innerHeight
  })
})
</script>

<style scoped>
.checker-container {
  width: 100vw;
  height: 100vh;
  position: relative;
  overflow: hidden;
  background: radial-gradient(ellipse at center, #1a1a2e 0%, #16213e 50%, #0a0a23 100%);
  animation: backgroundPulse 4s ease-in-out infinite alternate;
}

@keyframes backgroundPulse {
  0% { filter: brightness(0.8); }
  100% { filter: brightness(1.2); }
}
.bg-canvas {
  position: absolute;
  left: 0; top: 0;
  width: 100vw;
  height: 100vh;
  z-index: 1;
  pointer-events: none;
}
.main-content {
  position: absolute;
  z-index: 2;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  background: rgba(20, 24, 40, 0.85);
  border-radius: 24px;
  box-shadow: 0 8px 32px rgba(0,0,0,0.4), 0 0 60px rgba(64, 158, 255, 0.1);
  padding: 48px 40px 32px 40px;
  min-width: 420px;
  text-align: center;
  color: #fff;
  backdrop-filter: blur(8px);
  border: 1px solid rgba(64, 158, 255, 0.2);
  animation: containerFloat 3s ease-in-out infinite, borderGlow 2s ease-in-out infinite;
}

@keyframes containerFloat {
  0%, 100% { transform: translate(-50%, -50%); }
  50% { transform: translate(-50%, -52%); }
}
.pc-icon {
  margin-bottom: 12px;
  animation: float 2.5s ease-in-out infinite;
}
@keyframes float {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-16px); }
}
.title {
  font-size: 2.2rem;
  font-weight: 700;
  letter-spacing: 2px;
  margin-bottom: 10px;
  background: linear-gradient(90deg, #409EFF, #67C23A, #FFD700, #409EFF);
  background-clip: text;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}
.desc {
  font-size: 1.1rem;
  color: #b3c0d1;
  margin-bottom: 32px;
}
.detect-btn {
  font-size: 1.3rem;
  padding: 0 48px;
  height: 54px;
  margin-bottom: 32px;
  box-shadow: 0 4px 16px rgba(64, 158, 255, 0.3);
  transition: all 0.3s ease;
  background: linear-gradient(45deg, #409EFF, #5dade2);
  border: none;
  position: relative;
  overflow: hidden;
  animation: detectPulse 2s ease-in-out infinite;
}

.detect-btn::before {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255,255,255,0.2), transparent);
  transition: left 0.5s;
}

.detect-btn:hover::before {
  left: 100%;
}

.detect-btn:hover {
  box-shadow: 0 8px 32px rgba(64, 158, 255, 0.5);
  transform: translateY(-2px);
}
.result-box {
  margin: 24px 0 0 0;
  font-size: 1.3rem;
  color: #67C23A;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 12px;
  font-weight: bold;
  letter-spacing: 1px;
  animation: resultGlow 1.5s ease-in-out infinite;
}
.fade-enter-active, .fade-leave-active {
  transition: opacity 0.5s;
}
.fade-enter-from, .fade-leave-to {
  opacity: 0;
}
.footer {
  margin-top: 36px;
  color: #b3c0d1;
  font-size: 0.95rem;
  opacity: 0.7;
}

/* 扫描动画 */
@keyframes scanMove {
  0% { top: 0; opacity: 0; }
  50% { opacity: 1; }
  100% { top: 100%; opacity: 0; }
}

/* 按钮脉冲动画 */
@keyframes detectPulse {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.05); box-shadow: 0 0 20px rgba(64, 158, 255, 0.8); }
}

/* 结果框闪烁动画 */
@keyframes resultGlow {
  0%, 100% { box-shadow: 0 0 10px rgba(103, 194, 58, 0.3); }
  50% { box-shadow: 0 0 30px rgba(103, 194, 58, 0.8); }
}

/* 添加发光边框动画 */
@keyframes borderGlow {
  0%, 100% { border-color: rgba(64, 158, 255, 0.2); }
  50% { border-color: rgba(64, 158, 255, 0.8); }
}

@media (max-width: 600px) {
  .main-content {
    min-width: 90vw;
    padding: 24px 8px 16px 8px;
  }
  .title {
    font-size: 1.3rem;
  }
  .desc {
    font-size: 0.95rem;
  }
  .detect-btn {
    font-size: 1rem;
    height: 40px;
    padding: 0 18px;
  }
}
</style>
