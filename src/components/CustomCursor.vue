<template>
  <div
    v-if="fine"
    class="cur-root"
    :class="['t-' + theme, kind ? 'k-' + kind : '', { on: theme !== 'off' && visible }]"
    aria-hidden="true"
  >
    <canvas ref="cv" class="cur-trail"></canvas>

    <!-- ring utama (lag halus di belakang pointer) -->
    <div ref="ringEl" class="cur-ring">
      <!-- FASE MERAH: sci-fi targeting reticle -->
      <svg class="cur-svg cur-red" viewBox="-32 -32 64 64">
        <circle class="c-main" r="17" />
        <circle class="c-dash" r="22" />
        <path class="c-tick" d="M0 -31 V-22 M0 31 V22 M-31 0 H-22 M31 0 H22" />
        <path class="c-brk" d="M-24 -14 V-24 H-14 M14 -24 H24 V-14 M24 14 V24 H14 M-14 24 H-24 V14" />
      </svg>

      <!-- FASE HIJAU: eco-digital reticle + leaf core -->
      <svg class="cur-svg cur-green" viewBox="-32 -32 64 64">
        <circle class="g-main" r="16" />
        <g class="g-orb">
          <circle class="g-dash" r="24" />
          <path class="g-arc" d="M0 -24 A24 24 0 0 1 20.8 -12" />
        </g>
        <path class="g-leaf" d="M0 -8 C7 -4 7 4 0 9 C-7 4 -7 -4 0 -8Z" />
        <path class="g-vein" d="M0 -3 V7" />
      </svg>
    </div>

    <!-- titik tepat di posisi pointer -->
    <div ref="dotEl" class="cur-dot"></div>

    <!-- panah horizontal saat hover slider (fase hijau) -->
    <div ref="adjEl" class="cur-adjust">
      <svg viewBox="0 0 72 28">
        <path class="arr arr-l" d="M16 14 L28 6 V22 Z" />
        <path class="arr arr-r" d="M56 14 L44 6 V22 Z" />
        <path class="adj-line" d="M32 14 H40 M4 5 V23 M68 5 V23" />
      </svg>
    </div>

    <!-- label mikro -->
    <div ref="tagEl" class="cur-tag" :class="{ show: !!label }">{{ label }}</div>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'

const fine =
  typeof window !== 'undefined' &&
  window.matchMedia('(hover: hover) and (pointer: fine)').matches
const reduce =
  typeof window !== 'undefined' &&
  window.matchMedia('(prefers-reduced-motion: reduce)').matches

const theme = ref('off')
const kind = ref('')
const label = ref('')
const visible = ref(false)

const cv = ref(null)
const ringEl = ref(null)
const dotEl = ref(null)
const adjEl = ref(null)
const tagEl = ref(null)

const SEL = 'button, a[href], [role="button"], input, select, textarea, summary, [data-cursor]'

let mx = -100, my = -100, pmx = -100, pmy = -100
let rx = -100, ry = -100, s = 1
let hoverEl = null
let raf = null, last = 0, lastPick = 0
let ctx = null, dpr = 1, W = 0, H = 0
let mo = null, glitchT = null
const parts = []

const rand = (a, b) => a + Math.random() * (b - a)

function readTheme() {
  const t = document.documentElement.dataset.cursorTheme
  theme.value = t === 'red' || t === 'green' ? t : 'off'
  parts.length = 0
  hoverEl = null
  kind.value = ''
  label.value = ''
  if (ctx) ctx.clearRect(0, 0, W, H)
}

function resize() {
  W = window.innerWidth
  H = window.innerHeight
  dpr = Math.min(2, window.devicePixelRatio || 1)
  cv.value.width = W * dpr
  cv.value.height = H * dpr
  ctx.setTransform(dpr, 0, 0, dpr, 0, 0)
}

// cek elemen di bawah kursor (tiap ~90ms, sekaligus menangani elemen yang muncul/hilang)
function pick() {
  let el = document.elementFromPoint(mx, my)
  el = el && el.closest ? el.closest(SEL) : null
  if (el && (el.disabled || el.getAttribute('aria-disabled') === 'true')) el = null
  hoverEl = el

  let k = '', l = ''
  if (el) {
    const custom = el.dataset.cursorLabel
    if (theme.value === 'green') {
      if (el.matches('input[type="range"], [data-cursor="adjust"]')) {
        k = 'adjust'; l = custom || '[ ADJUST ]'
      } else {
        k = 'snap'; l = custom || ''
      }
    } else {
      k = 'target'
      l = custom || (el.matches('.node') ? '[ INSPECT ]' : '[ TARGET ]')
    }
  }
  if (k !== kind.value) kind.value = k
  if (l !== label.value) label.value = l
}

// ---------- partikel ----------
function spawn(x, y) {
  if (parts.length > 170) return null
  const green = theme.value === 'green'
  const r = Math.random()
  let p
  if (!green) {
    if (r < 0.72) {
      p = { type: 'ember', x: x + rand(-3, 3), y: y + rand(-3, 3),
        vx: rand(-0.02, 0.03), vy: rand(-0.07, -0.02),
        max: rand(600, 1100), sz: rand(1, 2.6),
        col: Math.random() < 0.5 ? '255,69,0' : '255,179,71' }
    } else {
      p = { type: 'ash', x: x + rand(-4, 4), y: y + rand(-4, 4),
        vx: rand(-0.015, 0.015), vy: rand(-0.03, -0.008),
        max: rand(900, 1500), sz: rand(2, 4), col: '150,140,130' }
    }
  } else if (r < 0.75) {
    p = { type: 'dust', x: x + rand(-4, 4), y: y + rand(-4, 4),
      vx: rand(-0.02, 0.02), vy: rand(-0.035, 0.01),
      max: rand(800, 1400), sz: rand(1, 2.4),
      col: Math.random() < 0.6 ? '0,255,136' : '56,189,248' }
  } else {
    p = { type: 'leaf', x: x + rand(-4, 4), y: y + rand(-4, 4),
      vx: rand(-0.03, 0.03), vy: rand(-0.02, 0.03),
      max: rand(1100, 1800), sz: rand(1.6, 2.6),
      rot: rand(0, 6.28), vr: rand(-0.004, 0.004), col: '0,255,136' }
  }
  p.life = 0
  p.seed = Math.random() * 1000
  parts.push(p)
  return p
}

function burst(x, y) {
  for (let i = 0; i < 10; i++) {
    const p = spawn(x, y)
    if (!p) return
    const a = Math.random() * Math.PI * 2
    const v = rand(0.06, 0.2)
    p.vx = Math.cos(a) * v
    p.vy = Math.sin(a) * v
  }
}

function drawParticles(dt) {
  ctx.clearRect(0, 0, W, H)
  for (let i = parts.length - 1; i >= 0; i--) {
    const p = parts[i]
    p.life += dt
    if (p.life >= p.max) { parts.splice(i, 1); continue }
    const u = p.life / p.max
    let a = 1 - u
    p.x += p.vx * dt
    p.y += p.vy * dt

    if (p.type === 'ember') {
      ctx.globalCompositeOperation = 'lighter'
      ctx.fillStyle = `rgba(${p.col},${a * 0.18})`
      ctx.beginPath(); ctx.arc(p.x, p.y, p.sz * 3.2, 0, 6.283); ctx.fill()
      ctx.fillStyle = `rgba(${p.col},${a})`
      ctx.beginPath(); ctx.arc(p.x, p.y, p.sz * (1 - u * 0.5), 0, 6.283); ctx.fill()
    } else if (p.type === 'ash') {
      ctx.globalCompositeOperation = 'source-over'
      ctx.fillStyle = `rgba(${p.col},${a * 0.35})`
      ctx.beginPath(); ctx.arc(p.x, p.y, p.sz * (1 + u), 0, 6.283); ctx.fill()
    } else if (p.type === 'dust') {
      p.x += Math.sin((p.life + p.seed) / 260) * 0.03 * dt
      a *= 0.6 + 0.4 * Math.sin(p.life / 80 + p.seed)
      ctx.globalCompositeOperation = 'lighter'
      ctx.fillStyle = `rgba(${p.col},${a * 0.2})`
      ctx.beginPath(); ctx.arc(p.x, p.y, p.sz * 3.4, 0, 6.283); ctx.fill()
      ctx.fillStyle = `rgba(${p.col},${a})`
      ctx.beginPath(); ctx.arc(p.x, p.y, p.sz, 0, 6.283); ctx.fill()
    } else {
      p.rot += p.vr * dt
      p.x += Math.sin((p.life + p.seed) / 400) * 0.02 * dt
      ctx.globalCompositeOperation = 'lighter'
      ctx.save()
      ctx.translate(p.x, p.y)
      ctx.rotate(p.rot)
      ctx.fillStyle = `rgba(${p.col},${a * 0.75})`
      ctx.beginPath(); ctx.ellipse(0, 0, p.sz * 2.2, p.sz, 0, 0, 6.283); ctx.fill()
      ctx.restore()
    }
  }
  ctx.globalCompositeOperation = 'source-over'
}

// ---------- loop utama ----------
function loop(t) {
  raf = requestAnimationFrame(loop)
  const dt = Math.min(50, t - last || 16)
  last = t
  if (theme.value === 'off') return

  if (t - lastPick > 90) { lastPick = t; pick() }
  if (hoverEl && !hoverEl.isConnected) hoverEl = null

  // target ring
  let px = mx, py = my, sc = 1, k = 0.22
  const kd = kind.value
  if (hoverEl) {
    const r = hoverEl.getBoundingClientRect()
    const cx = r.left + r.width / 2
    const cy = r.top + r.height / 2
    if (kd === 'snap') {
      px = cx; py = cy; sc = 0.42; k = 0.3
    } else if (kd === 'target') {
      sc = 1.7
      if (r.width <= 90 && r.height <= 90) { px = cx; py = cy; k = 0.3 }
    } else if (kd === 'adjust') {
      sc = 0.6
    }
  }
  rx += (px - rx) * k
  ry += (py - ry) * k
  s += (sc - s) * 0.22

  ringEl.value.style.transform = `translate3d(${rx}px,${ry}px,0) translate(-50%,-50%) scale(${s})`
  dotEl.value.style.transform = `translate3d(${mx}px,${my}px,0) translate(-50%,-50%)`
  adjEl.value.style.transform = `translate3d(${mx}px,${my}px,0) translate(-50%,-50%)`

  // label
  if (kd === 'adjust') {
    tagEl.value.style.transform = `translate3d(${mx}px,${my + 26}px,0) translateX(-50%)`
  } else {
    const off = 32 * s + 6
    const flip = rx > W - 170
    tagEl.value.style.transform = flip
      ? `translate3d(${rx - off}px,${ry - 9}px,0) translateX(-100%)`
      : `translate3d(${rx + off}px,${ry - 9}px,0)`
  }

  // jejak partikel + glitch
  const ox = pmx, oy = pmy
  const dx = mx - ox, dy = my - oy
  const sp = Math.hypot(dx, dy)
  pmx = mx; pmy = my
  if (visible.value && !reduce && sp > 0.5) {
    const n = Math.min(4, 1 + ((sp / 14) | 0))
    for (let i = 1; i <= n; i++) spawn(ox + (dx * i) / n, oy + (dy * i) / n)
  }
  if (theme.value === 'red' && sp > 26 && !glitchT) {
    ringEl.value.classList.add('glitch')
    glitchT = setTimeout(() => {
      ringEl.value?.classList.remove('glitch')
      glitchT = null
    }, 150)
  }
  drawParticles(dt)
}

// ---------- event ----------
function onMove(e) {
  mx = e.clientX; my = e.clientY
  if (!visible.value) { visible.value = true; rx = mx; ry = my; pmx = mx; pmy = my }
}
function onDown(e) {
  if (theme.value === 'off') return
  ringEl.value?.classList.add('press')
  if (!reduce) burst(e.clientX, e.clientY)
}
function onUp() { ringEl.value?.classList.remove('press') }
function onLeave() { visible.value = false }
function onEnter() { visible.value = true }

onMounted(() => {
  if (!fine) return
  ctx = cv.value.getContext('2d')
  resize()
  readTheme()
  mo = new MutationObserver(readTheme)
  mo.observe(document.documentElement, { attributes: true, attributeFilter: ['data-cursor-theme'] })
  window.addEventListener('resize', resize)
  window.addEventListener('mousemove', onMove, { passive: true })
  window.addEventListener('mousedown', onDown)
  window.addEventListener('mouseup', onUp)
  document.documentElement.addEventListener('mouseleave', onLeave)
  document.documentElement.addEventListener('mouseenter', onEnter)
  raf = requestAnimationFrame(loop)
})

onBeforeUnmount(() => {
  if (raf) cancelAnimationFrame(raf)
  clearTimeout(glitchT)
  mo?.disconnect()
  window.removeEventListener('resize', resize)
  window.removeEventListener('mousemove', onMove)
  window.removeEventListener('mousedown', onDown)
  window.removeEventListener('mouseup', onUp)
  document.documentElement.removeEventListener('mouseleave', onLeave)
  document.documentElement.removeEventListener('mouseenter', onEnter)
})
</script>

<!-- global: sembunyikan kursor bawaan hanya saat tema aktif (dan hanya di mouse/trackpad) -->
<style>
@media (hover: hover) and (pointer: fine) {
  html[data-cursor-theme='red'],
  html[data-cursor-theme='red'] *,
  html[data-cursor-theme='green'],
  html[data-cursor-theme='green'] * {
    cursor: none !important;
  }
}
</style>

<style scoped>
.cur-root {
  position: fixed;
  inset: 0;
  z-index: 2147483647;
  pointer-events: none;
  opacity: 0;
  transition: opacity 0.25s ease;
  --c: #ff4500;
  --c2: #ffb347;
}
.cur-root.on { opacity: 1; }
.t-green { --c: #00ff88; --c2: #38bdf8; }

.cur-trail { position: absolute; inset: 0; width: 100%; height: 100%; }

.cur-ring, .cur-dot, .cur-adjust, .cur-tag {
  position: absolute;
  left: 0;
  top: 0;
  will-change: transform;
}
.cur-ring { width: 64px; height: 64px; transition: opacity 0.15s ease; }
.k-adjust .cur-ring { opacity: 0; }

.cur-dot {
  width: 4px;
  height: 4px;
  border-radius: 50%;
  background: var(--c);
  box-shadow: 0 0 8px 2px var(--c);
}
.k-adjust .cur-dot { opacity: 0; }

/* ---------- svg umum ---------- */
.cur-svg {
  display: none;
  width: 100%;
  height: 100%;
  overflow: visible;
  filter: drop-shadow(0 0 4px var(--c)) drop-shadow(0 0 10px color-mix(in srgb, var(--c) 55%, transparent));
  transition: transform 0.18s ease;
}
.t-red .cur-red,
.t-green .cur-green { display: block; }
.cur-ring.press .cur-svg { transform: scale(0.8); }

.cur-svg * {
  fill: none;
  stroke: var(--c);
  stroke-width: 1.2;
  stroke-linecap: round;
  stroke-linejoin: round;
  transform-box: fill-box;
  transform-origin: center;
}

/* ---------- FASE MERAH ---------- */
.c-main { opacity: 0.9; transition: stroke-width 0.2s; }
.c-dash { stroke-dasharray: 3 6; opacity: 0.55; animation: curSpin 14s linear infinite; }
.c-tick { transition: opacity 0.2s; }
.c-brk  { opacity: 0; }
.k-target .c-main { stroke-width: 1.8; }
.k-target .c-tick { opacity: 0.3; }
.k-target .c-brk  { opacity: 1; animation: curLock 0.28s cubic-bezier(0.2, 0.9, 0.3, 1.2) both; }
@keyframes curLock {
  from { transform: scale(1.7) rotate(45deg); opacity: 0; }
  to   { transform: none; opacity: 1; }
}
@keyframes curSpin { to { transform: rotate(360deg); } }

/* glitch tipis saat kursor digerakkan cepat */
.cur-ring.glitch .cur-svg { animation: curGlitch 0.14s steps(2) 1; }
@keyframes curGlitch {
  0%   { filter: drop-shadow(3px 0 rgba(0, 255, 255, 0.85)) drop-shadow(-3px 0 rgba(255, 0, 60, 0.9)); transform: skewX(-8deg); }
  50%  { filter: drop-shadow(-2px 0 rgba(0, 255, 255, 0.85)) drop-shadow(2px 0 rgba(255, 0, 60, 0.9)); transform: skewX(6deg) translateX(2px); }
  100% { transform: none; }
}

/* ---------- FASE HIJAU ---------- */
.g-main { opacity: 0.9; transition: stroke-width 0.2s; }
.g-dash { stroke-dasharray: 2 7; opacity: 0.5; }
.g-arc  { stroke: var(--c2); stroke-width: 1.6; }
.g-orb  { animation: curSpin 6s linear infinite; }
.g-leaf { fill: var(--c); fill-opacity: 0.85; stroke: none; animation: leafPulse 2.4s ease-in-out infinite; }
.g-vein { stroke: #04170d; stroke-width: 0.9; }
.k-snap .g-main { stroke-width: 2; }
.k-snap .g-orb  { animation-duration: 2s; }
@keyframes leafPulse {
  0%, 100% { transform: scale(1); }
  50%      { transform: scale(1.18); }
}

/* panah [ ◄ ► ] saat hover slider */
.cur-adjust { width: 72px; height: 28px; opacity: 0; transition: opacity 0.15s ease; }
.k-adjust .cur-adjust { opacity: 1; }
.cur-adjust svg {
  width: 100%;
  height: 100%;
  overflow: visible;
  filter: drop-shadow(0 0 5px var(--c));
}
.arr { fill: var(--c); stroke: none; }
.adj-line { fill: none; stroke: var(--c2); stroke-width: 1.4; stroke-linecap: round; }
.arr-l { animation: nudgeL 0.9s ease-in-out infinite; }
.arr-r { animation: nudgeR 0.9s ease-in-out infinite; }
@keyframes nudgeL { 50% { transform: translateX(-4px); } }
@keyframes nudgeR { 50% { transform: translateX(4px); } }

/* ---------- label mikro ---------- */
.cur-tag {
  padding: 2px 7px;
  font-family: 'Courier New', monospace;
  font-size: 11px;
  font-weight: bold;
  letter-spacing: 0.14em;
  white-space: nowrap;
  color: var(--c);
  background: rgba(0, 0, 0, 0.65);
  border: 1px solid var(--c);
  text-shadow: 0 0 6px var(--c);
  opacity: 0;
  transition: opacity 0.15s ease;
}
.cur-tag.show { opacity: 1; }

@media (prefers-reduced-motion: reduce) {
  .cur-svg *, .cur-adjust *, .cur-ring.glitch .cur-svg { animation: none !important; }
}
</style>
