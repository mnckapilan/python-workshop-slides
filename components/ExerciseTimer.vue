<script setup>
import { ref, computed, onMounted, onUnmounted, nextTick } from 'vue'

const isStudentMode = import.meta.env.VITE_STUDENT_MODE === 'true'

// ── Timer state ─────────────────────────────────────────────
const DEFAULT_MIN = 3
const totalSecs  = ref(DEFAULT_MIN * 60)
const remaining  = ref(DEFAULT_MIN * 60)
const running    = ref(false)
const done       = ref(false)
const editing    = ref(false)
const editVal    = ref(String(DEFAULT_MIN))
const inputEl    = ref(null)
let ticker = null

// ── Timer control ────────────────────────────────────────────
function tick() {
  if (remaining.value <= 0) {
    clearInterval(ticker)
    ticker        = null
    running.value = false
    done.value    = true
    stopJazz()
    stopViz()
    playDing()
    return
  }
  remaining.value--
}

function toggle() {
  if (done.value) return
  running.value ? pause() : start()
}

function start() {
  if (remaining.value <= 0) return
  running.value = true
  done.value    = false
  ticker = setInterval(tick, 1000)
  startJazz()
  startViz()
}

function pause() {
  running.value = false
  clearInterval(ticker)
  ticker = null
  stopJazz()
  stopViz()
}

function reset() {
  pause()
  done.value      = false
  remaining.value = totalSecs.value
}

// ── Edit time ────────────────────────────────────────────────
function startEdit() {
  if (running.value) return
  editing.value = true
  editVal.value = String(Math.round(totalSecs.value / 60))
  nextTick(() => inputEl.value?.select())
}

function commitEdit() {
  editing.value   = false
  const mins      = Math.max(1, Math.min(99, parseInt(editVal.value) || DEFAULT_MIN))
  totalSecs.value = mins * 60
  remaining.value = mins * 60
  done.value      = false
}

// ── Display ─────────────────────────────────────────────────
const formatted = computed(() => {
  const m = Math.floor(remaining.value / 60)
  const s = remaining.value % 60
  return `${String(m).padStart(2, '0')}:${String(s).padStart(2, '0')}`
})

const progress = computed(() =>
  totalSecs.value ? ((totalSecs.value - remaining.value) / totalSecs.value) * 100 : 0
)

// ── Audio stream ─────────────────────────────────────────────
const STREAM_NAME = 'Bossa Beyond'
const STREAM_URL  = 'https://ice1.somafm.com/bossa-128-mp3'
let jazzEl = null

function startJazz() {
  if (!jazzEl) {
    jazzEl = new Audio(STREAM_URL)
    jazzEl.volume = 0.3
  }
  jazzEl.play().catch(() => {})
}

function stopJazz() {
  if (jazzEl) jazzEl.pause()
}

// ── Ding ─────────────────────────────────────────────────────
function playDing() {
  const ac = new AudioContext()
  ;[[880, 0.0], [659.25, 0.38]].forEach(([freq, delay]) => {
    const osc = ac.createOscillator()
    const env = ac.createGain()
    osc.connect(env)
    env.connect(ac.destination)
    osc.type = 'sine'
    osc.frequency.value = freq
    const t = ac.currentTime + delay
    env.gain.setValueAtTime(0.4, t)
    env.gain.exponentialRampToValueAtTime(0.001, t + 2.0)
    osc.start(t)
    osc.stop(t + 2.1)
  })
  setTimeout(() => ac.close(), 3000)
}

// ── Visualiser ───────────────────────────────────────────────
const canvasEl  = ref(null)
const BAR_COUNT = 18
const barH  = new Array(BAR_COUNT).fill(0.04)
const barT  = new Array(BAR_COUNT).fill(0.04)
let animId    = null
let targetId  = null

function updateTargets() {
  for (let i = 0; i < BAR_COUNT; i++) {
    // Bell-curve envelope — centre bars taller
    const pos = i / (BAR_COUNT - 1)
    const env = 1 - Math.abs(pos - 0.5) * 1.4
    barT[i] = Math.max(0.06, env * (0.35 + Math.random() * 0.65))
  }
}

function settleTargets() {
  for (let i = 0; i < BAR_COUNT; i++) barT[i] = 0.04
}

function drawViz() {
  animId = requestAnimationFrame(drawViz)
  const canvas = canvasEl.value
  if (!canvas) return
  const ctx = canvas.getContext('2d')
  const W   = canvas.width
  const H   = canvas.height
  const gap = 2
  const bW  = (W - gap * (BAR_COUNT - 1)) / BAR_COUNT

  ctx.clearRect(0, 0, W, H)

  const color = done.value ? '#00AB6C' : running.value ? '#0A6B64' : '#D6D2D7'

  for (let i = 0; i < BAR_COUNT; i++) {
    barH[i] += (barT[i] - barH[i]) * 0.14
    const bH = Math.max(2, barH[i] * H)
    const x  = Math.round(i * (bW + gap))
    ctx.fillStyle = color
    ctx.fillRect(x, H - bH, Math.max(1, Math.floor(bW)), bH)
  }
}

function startViz() {
  updateTargets()
  targetId = setInterval(updateTargets, 160)
}

function stopViz() {
  clearInterval(targetId)
  targetId = null
  settleTargets()
}

onMounted(() => {
  const canvas = canvasEl.value
  if (canvas) {
    canvas.width  = canvas.offsetWidth  || 148
    canvas.height = canvas.offsetHeight || 24
  }
  animId = requestAnimationFrame(drawViz)
})

// ── Cleanup ─────────────────────────────────────────────────
onUnmounted(() => {
  clearInterval(ticker)
  clearInterval(targetId)
  cancelAnimationFrame(animId)
  if (jazzEl) { jazzEl.pause(); jazzEl = null }
})
</script>

<template>
  <div v-if="!isStudentMode" class="et" :class="{ 'et-run': running, 'et-done': done }">
    <div class="et-label">Timer</div>
    <div
      v-if="!editing"
      class="et-time"
      @click="startEdit"
      :title="running ? '' : 'Click to change'"
    >{{ formatted }}</div>
    <input
      v-else
      ref="inputEl"
      class="et-input"
      v-model="editVal"
      type="number"
      min="1"
      max="99"
      @blur="commitEdit"
      @keydown.enter="commitEdit"
    />
    <div class="et-track">
      <div class="et-fill" :style="{ width: progress + '%' }"></div>
    </div>
    <canvas ref="canvasEl" class="et-viz"></canvas>
    <div class="et-np">
      <span class="et-np-dot"></span>
      <span class="et-np-name">{{ STREAM_NAME }}</span>
    </div>
    <div class="et-controls">
      <button class="et-btn et-play" @click="toggle" :disabled="done">
        <span v-if="running">&#9646;&#9646;</span>
        <span v-else>&#9654;</span>
      </button>
      <button class="et-btn et-reset" @click="reset">&#8635;</button>
    </div>
  </div>
</template>

<style scoped>
.et {
  position: absolute;
  bottom: 1.6rem;
  right: 2rem;
  width: 172px;
  background: var(--surface, #ffffff);
  border: 1px solid var(--border, #D6D2D7);
  border-radius: 10px;
  padding: 0.8rem 1rem 0.75rem;
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  font-family: var(--font-mono, 'Fira Code', monospace);
  box-shadow: 0 2px 14px rgba(0, 0, 0, 0.09);
  transition: border-color 0.25s, box-shadow 0.25s;
  user-select: none;
  z-index: 10;
}

.et-run  { border-color: #0A6B64; box-shadow: 0 2px 14px rgba(18, 148, 138, 0.18); }
.et-done { border-color: #00AB6C; box-shadow: 0 2px 14px rgba(0, 171, 108, 0.18); }

.et-label {
  font-family: 'CoFo Sans Semi-Mono', monospace;
  font-weight: 500;
  font-size: 0.62rem;
  letter-spacing: -0.02em;
  line-height: 1.2;
  text-transform: uppercase;
  color: #A89DAD;
}

.et-time {
  font-size: 1.95rem;
  font-weight: 700;
  text-align: center;
  letter-spacing: 0.04em;
  color: #321E37;
  line-height: 1;
  cursor: pointer;
  transition: color 0.25s;
}

.et-run  .et-time { color: #0A6B64; }
.et-done .et-time { color: #00AB6C; }

.et-input {
  font-family: var(--font-mono, 'Fira Code', monospace);
  font-size: 1.95rem;
  font-weight: 700;
  text-align: center;
  width: 100%;
  border: none;
  background: transparent;
  color: #321E37;
  outline: none;
  padding: 0;
}

.et-track {
  height: 3px;
  background: #D6D2D7;
  border-radius: 2px;
  overflow: hidden;
}

.et-fill {
  height: 100%;
  background: #0A6B64;
  border-radius: 2px;
  transition: width 1s linear, background 0.3s;
}

.et-done .et-fill {
  background: #00AB6C;
  width: 100% !important;
  transition: background 0.3s;
}

/* ── Visualiser ── */
.et-viz {
  width: 100%;
  height: 24px;
  display: block;
}

/* ── Now playing ── */
.et-np {
  display: flex;
  align-items: center;
  gap: 0.4rem;
}

.et-np-dot {
  width: 5px;
  height: 5px;
  border-radius: 50%;
  background: #D6D2D7;
  flex-shrink: 0;
  transition: background 0.3s;
}

.et-run .et-np-dot {
  background: #0A6B64;
  animation: et-pulse 1.4s ease-in-out infinite;
}

.et-done .et-np-dot {
  background: #00AB6C;
}

@keyframes et-pulse {
  0%, 100% { opacity: 1; }
  50%       { opacity: 0.3; }
}

.et-np-name {
  font-size: 0.68rem;
  color: #706273;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.et-run .et-np-name { color: #321E37; }

/* ── Controls ── */
.et-controls {
  display: flex;
  gap: 0.4rem;
}

.et-btn {
  padding: 0.28rem 0;
  background: #F8F5F0;
  border: 1px solid #D6D2D7;
  border-radius: 5px;
  cursor: pointer;
  color: #321E37;
  font-size: 0.85rem;
  line-height: 1.4;
  transition: background 0.1s;
}

.et-btn:hover:not(:disabled) { background: #D6D2D7; }
.et-btn:disabled { opacity: 0.35; cursor: default; }

.et-play  { flex: 1; }
.et-reset { flex: 0 0 2.2rem; font-size: 1rem; }
</style>
