<script setup>
import { ref } from 'vue'

const vars = [
  { name: 'song_title', value: '"Blinding Lights"', display: 'Blinding Lights', color: '#0A6B64', bg: '#E6F7F5' },
  { name: 'artist',     value: '"The Weeknd"',      display: 'The Weeknd',       color: '#706273', bg: '#EAE8EB' },
  { name: 'year',       value: '2019',              display: '2019',             color: '#00AB6C', bg: '#E6FBF3' },
]

const parts = [
  { type: 'literal', text: 'Now Playing: ' },
  { type: 'slot', varIdx: 0 },
  { type: 'literal', text: ' by ' },
  { type: 'slot', varIdx: 1 },
  { type: 'literal', text: ' (' },
  { type: 'slot', varIdx: 2 },
  { type: 'literal', text: ')' },
]

// phase: 'idle' | 'filling' | 'done'
const phase = ref('idle')
const filled = ref([false, false, false])
let timers = []

function evaluate() {
  if (phase.value === 'done') return reset()
  if (phase.value === 'filling') return
  phase.value = 'filling'
  filled.value = [false, false, false]
  vars.forEach((_, i) => {
    const t = setTimeout(() => {
      filled.value = filled.value.map((v, j) => j === i ? true : v)
      if (i === vars.length - 1) {
        setTimeout(() => { phase.value = 'done' }, 350)
      }
    }, i * 550)
    timers.push(t)
  })
}

function reset() {
  timers.forEach(clearTimeout)
  timers = []
  phase.value = 'idle'
  filled.value = [false, false, false]
}
</script>

<template>
  <div class="fv">
    <!-- Variable definitions -->
    <div class="fv-vars">
      <div v-for="v in vars" :key="v.name" class="fv-var" :style="{ borderColor: v.color, background: v.bg }">
        <span class="fv-var-name" :style="{ color: v.color }">{{ v.name }}</span>
        <span class="fv-var-eq">=</span>
        <span class="fv-var-val">{{ v.value }}</span>
      </div>
    </div>

    <!-- f-string template -->
    <div class="fv-template">
      <span class="fv-kw">f</span><span class="fv-quote">"</span>
      <template v-for="(part, i) in parts" :key="i">
        <span v-if="part.type === 'literal'" class="fv-lit">{{ part.text }}</span>
        <span
          v-else
          class="fv-slot"
          :class="{ 'fv-slot-filled': filled[part.varIdx] }"
          :style="{
            borderColor: vars[part.varIdx].color,
            background: filled[part.varIdx] ? vars[part.varIdx].bg : 'transparent',
            color: filled[part.varIdx] ? vars[part.varIdx].color : 'inherit',
          }"
        >
          <template v-if="!filled[part.varIdx]">
            {<span :style="{ color: vars[part.varIdx].color }">{{ vars[part.varIdx].name }}</span>}
          </template>
          <span v-else class="fv-slot-val">{{ vars[part.varIdx].display }}</span>
        </span>
      </template>
      <span class="fv-quote">"</span>
    </div>

    <!-- Output -->
    <div class="fv-output" :class="{ 'fv-output-ready': phase === 'done' }">
      <div class="fv-out-bar">output</div>
      <div class="fv-out-body">
        <span v-if="phase !== 'done'" class="fv-hint">press Evaluate to run…</span>
        <span v-else class="fv-result">
          <span class="fv-prompt">▶</span>
          Now Playing:&nbsp;<span :style="{ color: vars[0].color, fontWeight: 600 }">Blinding Lights</span>&nbsp;by&nbsp;<span :style="{ color: vars[1].color, fontWeight: 600 }">The Weeknd</span>&nbsp;(<span :style="{ color: vars[2].color, fontWeight: 600 }">2019</span>)
        </span>
      </div>
    </div>

    <!-- Controls -->
    <div class="fv-controls">
      <button
        class="fv-btn fv-btn-eval"
        :disabled="phase === 'filling'"
        @click="evaluate"
      >
        {{ phase === 'done' ? '↺ reset' : '▶ Evaluate' }}
      </button>
    </div>
  </div>
</template>

<style scoped>
.fv {
  display: flex;
  flex-direction: column;
  gap: 0.85rem;
  margin-top: 0.7rem;
}

/* ── Variables ── */
.fv-vars {
  display: flex;
  gap: 0.6rem;
  flex-wrap: wrap;
}

.fv-var {
  display: flex;
  align-items: center;
  gap: 0.45rem;
  padding: 0.35rem 0.85rem;
  border: 1.5px solid;
  border-radius: 7px;
  font-family: 'Roboto Mono', monospace;
  font-size: 0.88rem;
}

.fv-var-name { font-weight: 700; }
.fv-var-eq   { color: #A89DAD; }
.fv-var-val  { color: #321E37; }

/* ── Template ── */
.fv-template {
  display: flex;
  align-items: center;
  flex-wrap: wrap;
  gap: 0;
  padding: 0.65rem 1rem;
  background: #EAE8EB;
  border: 1px solid #D6D2D7;
  border-radius: 8px;
  font-family: 'Roboto Mono', monospace;
  font-size: 1rem;
  line-height: 1.9;
}

.fv-kw    { color: #0A6B64; font-weight: 700; margin-right: 1px; }
.fv-quote { color: #00AB6C; }
.fv-lit   { color: #00AB6C; }

.fv-slot {
  display: inline-flex;
  align-items: center;
  border: 1.5px solid;
  border-radius: 5px;
  padding: 0 0.3rem;
  transition: background 0.25s, color 0.25s;
  font-size: 0.95rem;
  line-height: 1.55;
}

.fv-slot-filled {
  animation: fv-pop 0.3s ease-out;
}

@keyframes fv-pop {
  0%   { transform: scale(1); }
  45%  { transform: scale(1.15); }
  100% { transform: scale(1); }
}

.fv-slot-val { font-weight: 700; }

/* ── Output ── */
.fv-output {
  border: 1px solid #D6D2D7;
  border-radius: 8px;
  overflow: hidden;
  background: #F8F5F0;
  font-family: 'Roboto Mono', monospace;
  transition: border-color 0.35s;
}

.fv-output-ready { border-color: #00AB6C; }

.fv-out-bar {
  padding: 0.28rem 0.85rem;
  background: #EAE8EB;
  border-bottom: 1px solid #D6D2D7;
  font-size: 0.72rem;
  color: #706273;
  letter-spacing: 0.05em;
}

.fv-out-body {
  padding: 0.55rem 0.9rem;
  min-height: 2.1rem;
  display: flex;
  align-items: center;
}

.fv-hint {
  color: #A89DAD;
  font-size: 0.85rem;
}

.fv-result {
  font-size: 0.92rem;
  color: #321E37;
  animation: fv-fadein 0.35s ease-out;
}

@keyframes fv-fadein {
  from { opacity: 0; transform: translateY(3px); }
  to   { opacity: 1; transform: translateY(0); }
}

.fv-prompt { color: #00AB6C; margin-right: 0.5rem; }

/* ── Controls ── */
.fv-controls { display: flex; }

.fv-btn {
  padding: 0.38rem 1.2rem;
  background: #EAE8EB;
  border: 1px solid #D6D2D7;
  border-radius: 6px;
  font-family: 'Roboto Mono', monospace;
  font-size: 0.88rem;
  color: #321E37;
  cursor: pointer;
  transition: background 0.1s;
}

.fv-btn:hover:not(:disabled) { background: #D6D2D7; }
.fv-btn:disabled { opacity: 0.35; cursor: default; }

.fv-btn-eval {
  background: #321E37;
  border-color: #321E37;
  color: #50FFEB;
  font-weight: 600;
}

.fv-btn-eval:hover:not(:disabled) { background: #4A3550; }
</style>
