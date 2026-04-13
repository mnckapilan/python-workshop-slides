<script setup>
import { ref, computed } from "vue";

const props = defineProps({
  filename: { type: String, default: "example.py" },
  // Static code shown at top (plain string, syntax-highlighted)
  setup: { type: String, default: "" },
  // Interactive rows: [{ code: "print(playlist[0])", output: "'Blinding Lights'" }]
  rows: { type: Array, default: () => [] },
  // Optional static code shown at bottom
  footer: { type: String, default: "" },
});

const revealed = ref(new Set());

function toggle(i) {
  const s = new Set(revealed.value);
  if (s.has(i)) s.delete(i);
  else s.add(i);
  revealed.value = s;
}

function reset() {
  revealed.value = new Set();
}

const anyRevealed = computed(() => revealed.value.size > 0);
</script>

<template>
  <div class="rb">
    <!-- Title bar -->
    <div class="rb-bar">
      <span class="rb-filename">{{ filename }}</span>
      <button v-if="anyRevealed" class="rb-reset" @click="reset">reset</button>
      <span v-else class="rb-reset-placeholder" />
    </div>

    <!-- Static setup block -->
    <div v-if="setup" class="rb-setup">
      <pre class="rb-pre">{{ setup }}</pre>
    </div>

    <!-- Divider -->
    <div v-if="setup && rows.length" class="rb-divider" />

    <!-- Interactive rows -->
    <div class="rb-rows">
      <div v-for="(row, i) in rows" :key="i" class="rb-row">
        <!-- Play / revealed button -->
        <button
          class="rb-btn"
          :class="{ 'rb-btn--done': revealed.has(i) }"
          @click="toggle(i)"
          :title="revealed.has(i) ? 'Hide output' : 'Show output'"
        >
          <span v-if="!revealed.has(i)">▶</span>
          <span v-else>▼</span>
        </button>

        <!-- Code expression -->
        <span class="rb-code">{{ row.code }}</span>

        <!-- Output (revealed on click) -->
        <transition name="rb-fade">
          <span v-if="revealed.has(i)" class="rb-output">
            <span class="rb-arrow">→</span>
            <span class="rb-result">{{ row.output }}</span>
          </span>
        </transition>
      </div>
    </div>

    <!-- Static footer block -->
    <div v-if="footer" class="rb-divider" />
    <div v-if="footer" class="rb-setup rb-footer">
      <pre class="rb-pre">{{ footer }}</pre>
    </div>
  </div>
</template>

<style scoped>
.rb {
  border: 1px solid #d3d3d4;
  border-radius: 8px;
  overflow: hidden;
  font-family: "Fira Code", monospace;
  background: #fafafa;
  font-size: 0.88rem;
  line-height: 1.62;
}

/* Title bar */
.rb-bar {
  display: flex;
  align-items: center;
  padding: 0.38rem 0.85rem;
  background: #ecedef;
  border-bottom: 1px solid #d3d3d4;
  gap: 0.7rem;
  min-height: 2rem;
}

.rb-filename {
  flex: 1;
  font-size: 0.78rem;
  color: #696c77;
  font-weight: 500;
  letter-spacing: 0.01em;
  user-select: none;
}

.rb-reset,
.rb-reset-placeholder {
  font-family: "Fira Code", monospace;
  font-size: 0.72rem;
  color: #696c77;
  background: none;
  border: 1px solid #d3d3d4;
  border-radius: 3px;
  padding: 0.08rem 0.5rem;
  cursor: pointer;
  line-height: 1.5;
  flex-shrink: 0;
}

.rb-reset:hover { background: #d3d3d4; }
.rb-reset-placeholder { visibility: hidden; pointer-events: none; }

/* Setup / footer static blocks */
.rb-setup {
  padding: 0.35rem 0.85rem;
  background: #fafafa;
}

.rb-footer {
  background: #f5f5f6;
}

.rb-pre {
  margin: 0;
  font-family: inherit;
  font-size: inherit;
  line-height: inherit;
  color: #383a42;
  white-space: pre;
}

.rb-divider {
  border-top: 1px dashed #e0e0e2;
}

/* Interactive rows */
.rb-rows {
  padding: 0.2rem 0;
}

.rb-row {
  display: flex;
  align-items: center;
  gap: 0.7rem;
  padding: 0.18rem 0.85rem;
  min-height: 1.9rem;
}

/* Play button */
.rb-btn {
  width: 1.55rem;
  height: 1.55rem;
  border-radius: 50%;
  border: 1.5px solid #4078f2;
  background: #fff;
  color: #4078f2;
  font-size: 0.6rem;
  cursor: pointer;
  flex-shrink: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: background 0.12s, color 0.12s;
  line-height: 1;
}

.rb-btn:hover {
  background: #4078f2;
  color: #fff;
}

.rb-btn--done {
  background: #4078f2;
  color: #fff;
}

/* Code expression */
.rb-code {
  color: #383a42;
  white-space: pre;
}

/* Output */
.rb-output {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.rb-arrow {
  color: #a0a1a7;
  font-size: 0.8rem;
}

.rb-result {
  color: #50a14f;
  font-weight: 500;
}

/* Fade transition */
.rb-fade-enter-active { transition: opacity 0.2s, transform 0.2s; }
.rb-fade-enter-from   { opacity: 0; transform: translateX(-6px); }
</style>
