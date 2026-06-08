<template>
  <div id="loader-wrapper" :class="store.imgLoadStatus ? 'loaded' : null">
    <!-- 粒子背景，绘制在最底层，幕布之下 -->
    <canvas ref="particleCanvas" class="particle-canvas" />

    <!-- 加载内容，z-index 在幕布之上，随幕布拉开而淡出 -->
    <div class="loader">
      <div class="logo-ring-wrapper">
        <svg class="ring-svg" viewBox="0 0 160 160" xmlns="http://www.w3.org/2000/svg">
          <circle cx="80" cy="80" r="72" fill="none" stroke="rgba(255,255,255,0.06)" stroke-width="1" />
          <circle
            cx="80" cy="80" r="72"
            fill="none"
            stroke="url(#arcGradient)"
            stroke-width="2"
            stroke-linecap="round"
            stroke-dasharray="120 332"
            class="ring-arc"
          />
          <circle cx="80" cy="80" r="54" fill="none" stroke="rgba(255,255,255,0.04)" stroke-width="1" />
          <circle
            cx="80" cy="80" r="54"
            fill="none"
            stroke="url(#arcGradientInner)"
            stroke-width="1.5"
            stroke-linecap="round"
            stroke-dasharray="60 280"
            class="ring-arc-inner"
          />
          <circle cx="80" cy="8"  r="3" fill="#fff"                  class="dot-outer" />
          <circle cx="80" cy="26" r="2" fill="rgba(255,255,255,0.6)" class="dot-inner" />
          <defs>
            <linearGradient id="arcGradient" x1="0%" y1="0%" x2="100%" y2="0%">
              <stop offset="0%"   stop-color="rgba(255,255,255,0)" />
              <stop offset="60%"  stop-color="rgba(255,255,255,0.9)" />
              <stop offset="100%" stop-color="rgba(255,255,255,0.3)" />
            </linearGradient>
            <linearGradient id="arcGradientInner" x1="0%" y1="0%" x2="100%" y2="0%">
              <stop offset="0%"   stop-color="rgba(180,200,255,0)" />
              <stop offset="70%"  stop-color="rgba(180,200,255,0.8)" />
              <stop offset="100%" stop-color="rgba(180,200,255,0.2)" />
            </linearGradient>
          </defs>
        </svg>
        <div class="logo-center">
          <div class="logo-icon">
            <div class="icon-bar bar1" />
            <div class="icon-bar bar2" />
            <div class="icon-bar bar3" />
          </div>
        </div>
      </div>

      <div class="loader-text">
        <span class="site-name">{{ siteName }}</span>
        <div class="progress-bar-wrapper">
          <div class="progress-bar">
            <div class="progress-fill" />
          </div>
        </div>
        <span class="tip">{{ tipText }}</span>
      </div>
    </div>

    <!-- 幕布：位于加载内容之上，向两侧滑开后首页内容自然透出 -->
    <div class="curtain curtain-left" />
    <div class="curtain curtain-right" />
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from "vue";
import { mainStore } from "@/store";

const store = mainStore();
const siteName = import.meta.env.VITE_SITE_NAME;

const tips = [
  "🤡 别看了在使劲儿加载了💢💢💢······",
];
const tipText = ref(tips[0]);
let tipTimer = null;

const particleCanvas = ref(null);
let animFrame = null;

function initParticles() {
  const canvas = particleCanvas.value;
  if (!canvas) return;
  const ctx = canvas.getContext("2d");
  canvas.width = window.innerWidth;
  canvas.height = window.innerHeight;

  const particles = Array.from({ length: 60 }, () => ({
    x: Math.random() * canvas.width,
    y: Math.random() * canvas.height,
    r: Math.random() * 1.2 + 0.2,
    alpha: Math.random() * 0.5 + 0.1,
    speed: Math.random() * 0.3 + 0.05,
    drift: (Math.random() - 0.5) * 0.15,
  }));

  function draw() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    particles.forEach((p) => {
      ctx.beginPath();
      ctx.arc(p.x, p.y, p.r, 0, Math.PI * 2);
      ctx.fillStyle = `rgba(255,255,255,${p.alpha})`;
      ctx.fill();
      p.y -= p.speed;
      p.x += p.drift;
      if (p.y < -4) {
        p.y = canvas.height + 4;
        p.x = Math.random() * canvas.width;
      }
    });
    animFrame = requestAnimationFrame(draw);
  }
  draw();
}

let tipIdx = 0;
onMounted(() => {
  initParticles();
  tipTimer = setInterval(() => {
    tipIdx = (tipIdx + 1) % tips.length;
    tipText.value = tips[tipIdx];
  }, 2200);
});

onUnmounted(() => {
  clearInterval(tipTimer);
  cancelAnimationFrame(animFrame);
});
</script>

<style lang="scss" scoped>
/* ─── 容器：fixed 铺满，无自身位移动画 ─────────────── */
#loader-wrapper {
  position: fixed;
  inset: 0;
  z-index: 999;
  /* 背景色与幕布同色，幕布拉开时中间区域透出页面内容 */
  background: transparent;
  pointer-events: none; /* 幕布打开过程中不拦截下层点击 */

  &.loaded {
    /* 等幕布完全滑走后（0.9s）再隐藏容器，彻底脱离渲染流 */
    visibility: hidden;
    transition: visibility 0s 1s;

    /* 加载内容：幕布开始移动时即刻淡出缩小，先于幕布完成 */
    .loader {
      opacity: 0;
      transform: scale(0.92);
      transition:
        opacity  0.3s 0s ease-out,
        transform 0.3s 0s ease-out;
    }

    /* 左幕布向左滑出 */
    .curtain-left {
      transform: translateX(-100%);
      transition: transform 0.9s 0.15s cubic-bezier(0.76, 0, 0.24, 1);
    }

    /* 右幕布向右滑出，与左幕布完全同步 */
    .curtain-right {
      transform: translateX(100%);
      transition: transform 0.9s 0.15s cubic-bezier(0.76, 0, 0.24, 1);
    }
  }
}

/* ─── 粒子画布：铺在最底层 ──────────────────────── */
.particle-canvas {
  position: absolute;
  inset: 0;
  pointer-events: none;
  opacity: 0.7;
  z-index: 0;
}

/* ─── 幕布 ──────────────────────────────────────── */
.curtain {
  position: absolute;
  top: 0;
  width: 50.5%; /* 微量重叠防中缝 */
  height: 100%;
  z-index: 2;
  pointer-events: auto;
  /*
   * 幕布背景与加载器背景一致。
   * 幕布向两侧滑开后，中间缺口处透出的是 fixed 容器
   * 下方的真实页面内容——实现随拉开进度动态露出首页。
   */
  background: radial-gradient(ellipse 160% 120% at 50% 40%, #1a1f2e 0%, #0d0f18 100%);

  &.curtain-left {
    left: 0;
    /* 内侧向右渐暗，增加层次感 */
    box-shadow: inset -60px 0 120px rgba(0, 0, 0, 0.55);
  }
  &.curtain-right {
    right: 0;
    box-shadow: inset 60px 0 120px rgba(0, 0, 0, 0.55);
  }
}

/* ─── 加载内容：叠在幕布之上 ───────────────────── */
.loader {
  position: absolute;
  inset: 0;
  z-index: 3;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  pointer-events: auto;
  /* 默认状态保持完整，由 .loaded 触发退出过渡 */
  transform: scale(1);
  transition: none;
}

/* ─── 环形动画 ──────────────────────────────────── */
.logo-ring-wrapper {
  position: relative;
  width: 250px;
  height: 250px;
}

.ring-svg {
  width: 100%;
  height: 100%;
  overflow: visible;
}

.ring-arc {
  transform-origin: 80px 80px;
  animation: arc-spin 2.4s linear infinite;
}

.ring-arc-inner {
  transform-origin: 80px 80px;
  animation: arc-spin-rev 1.6s linear infinite;
}

.dot-outer {
  transform-origin: 80px 80px;
  animation: arc-spin 2.4s linear infinite;
  filter: drop-shadow(0 0 4px #fff);
}

.dot-inner {
  transform-origin: 80px 80px;
  animation: arc-spin-rev 1.6s linear infinite;
  filter: drop-shadow(0 0 3px rgba(180, 200, 255, 0.9));
}

@keyframes arc-spin     { to { transform: rotate(360deg);  } }
@keyframes arc-spin-rev { to { transform: rotate(-360deg); } }

/* ─── 中心图标 ──────────────────────────────────── */
.logo-center {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
}

.logo-icon {
  display: flex;
  align-items: flex-end;
  gap: 10px;        /* 原 5px → 7px，间距同步放大 */
  height: 60px;    /* 原 28px → 40px */
}

.icon-bar {
  width: 8px;      /* 原 5px → 7px */
  border-radius: 4px;
  background: rgba(255, 255, 255, 0.9);
  animation: bar-pulse 1.4s ease-in-out infinite;
}

.bar1 { height: 22px; }   /* 原 12px */
.bar2 { height: 36px; }   /* 原 22px */
.bar3 { height: 28px; }   /* 原 16px */

@keyframes bar-pulse {
  0%, 100% { opacity: 0.3; transform: scaleY(0.6); }
  50%       { opacity: 1;   transform: scaleY(1);   }
}

/* ─── 文字 ──────────────────────────────────────── */
.loader-text {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-top: 30px;
  gap: 16px;
}

.site-name {
  font-size: 50px;
  font-weight: 700;
  letter-spacing: 0.25em;
  text-transform: uppercase;
  background: linear-gradient(
    90deg,
    #ff6b6b, #ffd93d, #6bcb77, #4d96ff, #c77dff, #ff6b6b
  );
  background-size: 300% auto;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  filter: drop-shadow(0 0 8px rgba(255, 255, 255, 0.3));
  animation: rainbow-flow 3s linear infinite;
}

@keyframes rainbow-flow {
  0%   { background-position: 0% center; }
  100% { background-position: 300% center; }
}

@keyframes text-shine {
  0%   { background-position: 200% center;  }
  100% { background-position: -200% center; }
}

/* ─── 进度扫描条 ─────────────────────────────────── */
.progress-bar-wrapper { width: 120px; }

.progress-bar {
  height: 1px;
  background: rgba(255, 255, 255, 0.12);
  border-radius: 1px;
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  width: 60%;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.9), transparent);
  animation: progress-scan 1.8s ease-in-out infinite;
}

@keyframes progress-scan {
  0%   { transform: translateX(-120%); }
  100% { transform: translateX(280%);  }
}

/* ─── Tip ────────────────────────────────────────── */
.tip {
  font-size: 20px;                  /* 13px → 18px */
  color: #00ffb3;                   /* 亮青绿，清爽显眼 */
  letter-spacing: 0.08em;
  font-weight: 400;
  filter: drop-shadow(0 0 6px rgba(0, 255, 179, 0.5)); /* 加发光 */
  animation: tip-fade 2.2s ease-in-out infinite;
}

@keyframes tip-fade {
  0%, 100% { opacity: 0.7; }
  50%       { opacity: 1;  }
}

@keyframes tip-fade {
  0%, 100% { opacity: 0.35; }
  50%       { opacity: 0.7;  }
}
</style>