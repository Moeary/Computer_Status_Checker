<template>
  <div class="checker-container">
    <canvas ref="bgCanvas" class="bg-canvas"></canvas>
    <div class="main-content" :data-device-type="currentDeviceInfo.type">
      <el-icon class="pc-icon" size="48" color="#409EFF">
        <Monitor v-if="currentDeviceInfo.type === 'desktop'" />
        <Iphone v-else-if="currentDeviceInfo.type === 'mobile'" />
        <Notebook v-else />
      </el-icon>
      <h1 class="title">{{ texts.title }}</h1>
      <p class="desc">{{ texts.description }}</p>
      <div class="device-info">
        <small>检测到设备: {{ currentDeviceInfo.os }}</small>
      </div>
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
          <span class="result-text">{{ texts.result }}</span>
        </div>
      </transition>
      <footer class="footer">
        {{ texts.footer }}
      </footer>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'
import { Monitor, CircleCheckFilled, Iphone, Notebook } from '@element-plus/icons-vue'
import gsap from 'gsap'

const loading = ref(false)
const showResult = ref(false)
const bgCanvas = ref(null)

// 设备检测函数
const detectDevice = () => {
  const userAgent = navigator.userAgent.toLowerCase()
  const platform = navigator.platform.toLowerCase()
  const maxTouchPoints = navigator.maxTouchPoints || 0
  
  // 更精确的移动设备检测
  if (/android/.test(userAgent)) {
    const isTablet = /tablet/.test(userAgent) || (maxTouchPoints > 1 && !/mobile/.test(userAgent))
    return { 
      type: isTablet ? 'tablet' : 'mobile', 
      os: 'Android', 
      device: isTablet ? '平板' : '手机' 
    }
  }
  
  if (/iphone|ipod/.test(userAgent)) {
    return { type: 'mobile', os: 'iOS', device: '手机' }
  }
  
  if (/ipad/.test(userAgent) || (navigator.platform === 'MacIntel' && maxTouchPoints > 1)) {
    return { type: 'tablet', os: 'iPadOS', device: '平板' }
  }
  
  // 检测桌面系统
  if (/mac/.test(platform) || /darwin/.test(userAgent)) {
    return { type: 'desktop', os: 'macOS', device: '电脑' }
  }
  
  if (/win/.test(platform)) {
    // 检测Windows版本
    let winVersion = 'Windows'
    if (/windows nt 10.0/.test(userAgent)) winVersion = 'Windows 10/11'
    else if (/windows nt 6.3/.test(userAgent)) winVersion = 'Windows 8.1'
    else if (/windows nt 6.2/.test(userAgent)) winVersion = 'Windows 8'
    else if (/windows nt 6.1/.test(userAgent)) winVersion = 'Windows 7'
    
    return { type: 'desktop', os: winVersion, device: '电脑' }
  }
  
  if (/linux/.test(platform) && !/android/.test(userAgent)) {
    // 检测Linux发行版
    let linuxDistro = 'Linux'
    if (/ubuntu/.test(userAgent)) linuxDistro = 'Ubuntu'
    else if (/debian/.test(userAgent)) linuxDistro = 'Debian'
    else if (/fedora/.test(userAgent)) linuxDistro = 'Fedora'
    else if (/centos/.test(userAgent)) linuxDistro = 'CentOS'
    
    return { type: 'desktop', os: linuxDistro, device: '电脑' }
  }
  
  // Chrome OS检测
  if (/cros/.test(userAgent)) {
    return { type: 'desktop', os: 'Chrome OS', device: '电脑' }
  }
  
  // 其他移动设备检测
  if (/mobile|tablet|touch|phone/.test(userAgent) || maxTouchPoints > 0) {
    return { type: 'mobile', os: '移动设备', device: '手机' }
  }
  
  // 默认为电脑
  return { type: 'desktop', os: '未知系统', device: '电脑' }
}

// 获取设备信息
const deviceInfo = detectDevice()

// 开发调试：手动设备切换（仅开发环境）
const debugMode = ref(false)
const manualDevice = ref(null)

// 切换调试模式
const toggleDebugMode = () => {
  debugMode.value = !debugMode.value
}

// 手动设置设备类型
const setManualDevice = (type, os, device) => {
  manualDevice.value = { type, os, device }
}

// 实际使用的设备信息（支持手动覆盖）
const currentDeviceInfo = computed(() => {
  return manualDevice.value || deviceInfo
})

// 动态文本计算
const texts = computed(() => {
  const device = currentDeviceInfo.value.device
  return {
    title: `${device}开机状态检测器`,
    description: `使用先进的大数据技术分析您的${device}状态`,
    result: `恭喜你，你的${device}当前为开机状态！`,
    footer: `© 2025 ${device}健康检测中心 | 使用先进的大数据技术`
  }
})

const startDetect = () => {
  loading.value = true
  showResult.value = false
  
  // 强化动效：按钮震动+发光
  gsap.to('.detect-btn', { scale: 1.1, yoyo: true, repeat: 5, duration: 0.1 })
    // 显示检测过程信息
  console.log(`🔍 正在检测${currentDeviceInfo.value.device}状态...`)
  console.log(`📱 检测到设备类型: ${currentDeviceInfo.value.type}`)
  console.log(`💻 操作系统: ${currentDeviceInfo.value.os}`)
  
  setTimeout(() => {
    loading.value = false
    showResult.value = true
    gsap.fromTo('.result-box', 
      { scale: 0.5, opacity: 0, rotationY: 180 }, 
      { scale: 1, opacity: 1, rotationY: 0, duration: 0.8, ease: 'back.out(1.7)' }
    )
    // 粒子爆发动画
    triggerParticles()
    
    // 成功日志
    console.log(`✅ ${currentDeviceInfo.value.device}状态检测完成！`)
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
  
  function resizeCanvas() {
    canvas.width = window.innerWidth
    canvas.height = window.innerHeight
  }
  
  resizeCanvas()
  
  // 使用Canvas 2D代替Three.js避免兼容性问题
  const ctx = canvas.getContext('2d')
  const particles = []
  
  // 创建背景粒子
  function createParticles() {
    particles.length = 0
    const particleCount = Math.min(80, Math.floor((canvas.width * canvas.height) / 20000))
    
    for (let i = 0; i < particleCount; i++) {
      particles.push({
        x: Math.random() * canvas.width,
        y: Math.random() * canvas.height,
        vx: (Math.random() - 0.5) * 0.5,
        vy: (Math.random() - 0.5) * 0.5,
        size: Math.random() * 3 + 1,
        color: `hsl(${200 + Math.random() * 100}, 70%, 60%)`
      })
    }
  }
  
  createParticles()
  
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
    resizeCanvas()
    createParticles()
  })
})
</script>

<style scoped>
.checker-container {
  width: 100vw;
  height: 100vh;
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  overflow: hidden;
  background: radial-gradient(ellipse at center, #1a1a2e 0%, #16213e 50%, #0a0a23 100%);
  animation: backgroundPulse 4s ease-in-out infinite alternate;
  display: flex;
  align-items: center;
  justify-content: center;
}

@keyframes backgroundPulse {
  0% { filter: brightness(0.8); }
  100% { filter: brightness(1.2); }
}
.bg-canvas {
  position: fixed;
  left: 0; 
  top: 0;
  width: 100vw;
  height: 100vh;
  z-index: 1;
  pointer-events: none;
}
.main-content {
  position: relative;
  z-index: 2;
  background: rgba(20, 24, 40, 0.85);
  border-radius: 24px;
  box-shadow: 0 8px 32px rgba(0,0,0,0.4), 0 0 60px rgba(64, 158, 255, 0.1);
  padding: 48px 40px 32px 40px;
  min-width: 420px;
  max-width: 90vw;
  text-align: center;
  color: #fff;
  backdrop-filter: blur(8px);
  border: 1px solid rgba(64, 158, 255, 0.2);
  animation: containerFloat 3s ease-in-out infinite, borderGlow 2s ease-in-out infinite;
}

@keyframes containerFloat {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-8px); }
}
.pc-icon {
  margin-bottom: 12px;
  animation: float 2.5s ease-in-out infinite;
  transition: all 0.3s ease;
}
.pc-icon:hover {
  transform: scale(1.1);
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
.device-info {
  margin-bottom: 20px;
  padding: 8px 16px;
  background: rgba(64, 158, 255, 0.1);
  border-radius: 12px;
  border: 1px solid rgba(64, 158, 255, 0.2);
}

.device-info small {
  color: #409EFF;
  font-size: 0.9rem;
  font-weight: 500;
  letter-spacing: 0.5px;
}

/* 根据设备类型动态主题 */
.device-mobile {
  --primary-color: #00d4aa;
  --secondary-color: #00b894;
}

.device-tablet {
  --primary-color: #6c5ce7;
  --secondary-color: #a29bfe;
}

.device-desktop {
  --primary-color: #409EFF;
  --secondary-color: #5dade2;
}

.main-content[data-device-type="mobile"] .detect-btn {
  background: linear-gradient(45deg, #00d4aa, #00b894);
}

.main-content[data-device-type="tablet"] .detect-btn {
  background: linear-gradient(45deg, #6c5ce7, #a29bfe);
}

.main-content[data-device-type="mobile"] .device-info {
  background: rgba(0, 212, 170, 0.1);
  border-color: rgba(0, 212, 170, 0.2);
}

.main-content[data-device-type="mobile"] .device-info small {
  color: #00d4aa;
}

.main-content[data-device-type="tablet"] .device-info {
  background: rgba(108, 92, 231, 0.1);
  border-color: rgba(108, 92, 231, 0.2);
}

.main-content[data-device-type="tablet"] .device-info small {
  color: #6c5ce7;
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

@media (max-width: 768px) {
  .main-content {
    min-width: 85vw;
    max-width: 90vw;
    padding: 32px 24px 24px 24px;
    margin: 0 20px;
  }
  .title {
    font-size: 1.6rem;
    letter-spacing: 1px;
  }
  .desc {
    font-size: 0.95rem;
    margin-bottom: 24px;
  }
  .detect-btn {
    font-size: 1.1rem;
    height: 48px;
    padding: 0 32px;
  }
  .result-box {
    font-size: 1.1rem;
    gap: 8px;
  }
}

@media (max-width: 480px) {
  .main-content {
    min-width: 90vw;
    max-width: 95vw;
    padding: 24px 16px 20px 16px;
    margin: 0 10px;
  }
  .title {
    font-size: 1.3rem;
  }
  .desc {
    font-size: 0.9rem;
  }
  .detect-btn {
    font-size: 1rem;
    height: 44px;
    padding: 0 24px;
  }
  .pc-icon {
    font-size: 36px !important;
  }
}

@media (min-width: 1440px) {
  .main-content {
    min-width: 480px;
    max-width: 600px;
    padding: 56px 48px 40px 48px;
  }
  .title {
    font-size: 2.6rem;
  }
  .desc {
    font-size: 1.2rem;
  }
}

@media (min-width: 1920px) {
  .main-content {
    min-width: 560px;
    max-width: 700px;
    padding: 64px 56px 48px 56px;
  }
  .title {
    font-size: 3rem;
  }
  .desc {
    font-size: 1.3rem;
  }
  .detect-btn {
    font-size: 1.4rem;
    height: 60px;
    padding: 0 56px;
  }
}

/* 超宽屏和高分辨率优化 */
@media (min-width: 2560px) {
  .main-content {
    min-width: 640px;
    max-width: 800px;
    padding: 72px 64px 56px 64px;
  }
  .title {
    font-size: 3.5rem;
    letter-spacing: 3px;
  }
  .desc {
    font-size: 1.5rem;
    margin-bottom: 40px;
  }
  .detect-btn {
    font-size: 1.6rem;
    height: 68px;
    padding: 0 64px;
  }
  .result-box {
    font-size: 1.6rem;
    margin-top: 32px;
  }
}

/* 4K和8K屏幕优化 */
@media (min-width: 3840px) {
  .main-content {
    min-width: 800px;
    max-width: 1000px;
    padding: 96px 80px 72px 80px;
  }
  .title {
    font-size: 4rem;
    letter-spacing: 4px;
  }
  .desc {
    font-size: 1.8rem;
  }
  .detect-btn {
    font-size: 1.8rem;
    height: 80px;
    padding: 0 80px;
  }
  .pc-icon {
    font-size: 72px !important;
  }
}

/* 垂直屏幕优化 */
@media (orientation: portrait) and (max-height: 700px) {
  .main-content {
    padding: 20px 24px 16px 24px;
    max-height: 80vh;
    overflow-y: auto;
  }
  .title {
    font-size: 1.4rem;
    margin-bottom: 8px;
  }
  .desc {
    font-size: 0.9rem;
    margin-bottom: 20px;
  }
  .detect-btn {
    height: 40px;
    font-size: 1rem;
    margin-bottom: 20px;
  }
  .footer {
    margin-top: 20px;
    font-size: 0.8rem;
  }
}

/* 极小屏幕优化 */
@media (max-width: 320px) {
  .main-content {
    min-width: 95vw;
    padding: 20px 12px 16px 12px;
    margin: 0 5px;
  }
  .title {
    font-size: 1.1rem;
    letter-spacing: 0.5px;
  }
  .desc {
    font-size: 0.8rem;
  }
  .detect-btn {
    font-size: 0.9rem;
    height: 40px;
    padding: 0 20px;
  }
}
</style>
