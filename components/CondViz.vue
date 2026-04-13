<script setup>
import { ref, computed } from 'vue'

const bpm = ref(140)

const branch = computed(() => {
  if (bpm.value > 150) return 0
  if (bpm.value > 100) return 1
  return 2
})

const branches = computed(() => [
  {
    label: 'if',   check: 'bpm > 150', threshold: 150,
    result: '"High energy!"',    color: '#DE352D', bg: '#FFF0EE', border: '#F5ADAB',
    passes: bpm.value > 150,
  },
  {
    label: 'elif', check: 'bpm > 100', threshold: 100,
    result: '"Mid-tempo"',        color: '#12948A', bg: '#E6F7F5', border: '#A7FFF5',
    passes: bpm.value > 100,
    skipped: bpm.value > 150,
  },
  {
    label: 'else', check: null,
    result: '"Slow and relaxed"', color: '#00AB6C', bg: '#E6FBF3', border: '#80D4B0',
    passes: bpm.value <= 100,
    skipped: bpm.value > 100,
  },
])

const presets = [
  { name: 'Blinding Lights', bpm: 171 },
  { name: 'Levitating',      bpm: 103 },
  { name: 'Stay',            bpm:  90 },
  { name: 'Heat Waves',      bpm:  81 },
]
</script>

<template>
  <div class="cv">
    <!-- BPM input -->
    <div class="cv-top">
      <div class="cv-bpm-wrap">
        <span class="cv-code">song_bpm =</span>
        <div class="cv-bpm-display">{{ bpm }}</div>
        <input class="cv-slider" type="range" min="60" max="200" v-model.number="bpm" />
      </div>
      <div class="cv-presets">
        <button
          v-for="p in presets"
          :key="p.name"
          class="cv-preset"
          :class="{ 'cv-preset-active': bpm === p.bpm }"
          @click="bpm = p.bpm"
        >{{ p.name }} <span class="cv-preset-bpm">{{ p.bpm }}</span></button>
      </div>
    </div>

    <!-- Branches -->
    <div class="cv-branches">
      <div
        v-for="(b, i) in branches"
        :key="i"
        class="cv-branch"
        :class="{ 'cv-branch-active': branch === i, 'cv-branch-dim': branch !== i }"
        :style="branch === i ? { borderColor: b.border, background: b.bg } : {}"
      >
        <span class="cv-kw" :style="branch === i ? { color: b.color } : {}">{{ b.label }}</span>

        <!-- condition with live BPM substituted -->
        <span v-if="b.check" class="cv-condition" :class="{ 'cv-condition-active': branch === i }">
          <span class="cv-sub">{{ bpm }}</span>
          <span class="cv-op"> > {{ b.threshold }}</span>
        </span>
        <span v-else class="cv-condition cv-condition-else">(no condition)</span>

        <!-- true / false / skipped badge -->
        <span
          class="cv-badge"
          :class="b.skipped ? 'cv-badge-skip' : b.passes ? 'cv-badge-true' : 'cv-badge-false'"
        >{{ b.skipped ? 'skipped' : b.passes ? 'true ✓' : 'false ✗' }}</span>

        <span class="cv-arr" :style="branch === i ? { color: b.color } : {}">→</span>
        <span class="cv-result" :style="branch === i ? { color: b.color } : {}">{{ b.result }}</span>
      </div>
    </div>
  </div>
</template>

<style scoped>
.cv {
  display: flex;
  flex-direction: column;
  gap: 1.4rem;
  margin-top: 1rem;
}

/* ── Top row ── */
.cv-top {
  display: flex;
  align-items: center;
  gap: 2rem;
  flex-wrap: wrap;
}

.cv-bpm-wrap {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.cv-code {
  font-family: 'Fira Code', monospace;
  font-size: 1.05rem;
  color: #706273;
}

.cv-bpm-display {
  font-family: 'Fira Code', monospace;
  font-size: 2rem;
  font-weight: 700;
  color: #321E37;
  min-width: 3.5rem;
  text-align: center;
  background: #EAE8EB;
  border: 1px solid #D6D2D7;
  border-radius: 8px;
  padding: 0.1rem 0.7rem;
}

.cv-slider {
  -webkit-appearance: none;
  appearance: none;
  width: 180px;
  height: 5px;
  border-radius: 3px;
  background: #D6D2D7;
  outline: none;
  cursor: pointer;
}

.cv-slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 18px;
  height: 18px;
  border-radius: 50%;
  background: #12948A;
  cursor: pointer;
  box-shadow: 0 1px 4px rgba(18, 148, 138, 0.35);
}

/* ── Presets ── */
.cv-presets {
  display: flex;
  gap: 0.5rem;
  flex-wrap: wrap;
}

.cv-preset {
  padding: 0.3rem 0.75rem;
  background: #EAE8EB;
  border: 1px solid #D6D2D7;
  border-radius: 5px;
  font-family: 'Fira Code', monospace;
  font-size: 0.78rem;
  color: #706273;
  cursor: pointer;
  transition: background 0.1s, border-color 0.1s;
}

.cv-preset:hover { background: #D6D2D7; }

.cv-preset-active {
  background: #E6F7F5;
  border-color: #12948A;
  color: #12948A;
}

.cv-preset-bpm {
  font-weight: 700;
  margin-left: 0.25rem;
}

/* ── Branches ── */
.cv-branches {
  display: flex;
  flex-direction: column;
  gap: 0.6rem;
}

.cv-branch {
  display: flex;
  align-items: center;
  gap: 0.9rem;
  padding: 0.75rem 1.2rem;
  border: 1px solid #D6D2D7;
  border-radius: 8px;
  font-family: 'Fira Code', monospace;
  font-size: 1rem;
  background: #EAE8EB;
  transition: background 0.2s, border-color 0.2s, transform 0.2s, opacity 0.2s;
}

.cv-branch-active {
  transform: translateX(8px);
  box-shadow: 0 2px 12px rgba(0,0,0,0.08);
}

.cv-branch-dim {
  opacity: 0.38;
}

.cv-kw {
  font-weight: 700;
  color: #706273;
  min-width: 2.5rem;
  transition: color 0.2s;
}

.cv-check {
  color: #706273;
  flex-shrink: 0;
}

.cv-condition {
  font-family: 'Fira Code', monospace;
  font-size: 0.95rem;
  color: #A89DAD;
  background: #EAE8EB;
  border-radius: 5px;
  padding: 0.15rem 0.6rem;
  transition: background 0.2s, color 0.2s;
}

.cv-condition-active .cv-sub {
  color: #321E37;
  font-weight: 700;
}

.cv-condition-else {
  font-style: italic;
  font-size: 0.82rem;
}

.cv-badge {
  font-size: 0.78rem;
  font-weight: 700;
  letter-spacing: 0.03em;
  padding: 0.15rem 0.55rem;
  border-radius: 4px;
  flex-shrink: 0;
  transition: all 0.2s;
}

.cv-badge-true  { background: #E6FBF3; color: #00AB6C; border: 1px solid #80D4B0; }
.cv-badge-false { background: #FFF0EE; color: #DE352D; border: 1px solid #F5ADAB; }
.cv-badge-skip  { background: #EAE8EB; color: #A89DAD; border: 1px solid #D6D2D7; }

.cv-arr {
  color: #D6D2D7;
  font-size: 1.1rem;
  transition: color 0.2s;
}

.cv-result {
  font-weight: 600;
  color: #A89DAD;
  transition: color 0.2s;
}

@keyframes cv-pop {
  from { transform: scale(0.5); opacity: 0; }
  to   { transform: scale(1);   opacity: 1; }
}
</style>
