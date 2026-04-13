<script setup>
defineProps({
  file:   { type: String,  required: true },
  active: { type: Boolean, default: false },
})
</script>

<template>
  <div class="fh" :class="active ? 'fh-active' : 'fh-default'">
    <span class="fh-icon">{{ active ? '💻' : '📄' }}</span>
    <span class="fh-label">{{ file }}</span>
  </div>
</template>

<style scoped>
/* ── Shared ── */
.fh {
  position: absolute;
  display: flex;
  align-items: center;
  gap: 0.4rem;
  white-space: nowrap;
  z-index: 5;
  font-family: 'Roboto Mono', monospace;
}

/* ── Default: bottom-centre pill ── */
.fh-default {
  bottom: 1.6rem;
  left: 50%;
  transform: translateX(-50%);
  padding: 0.28rem 0.85rem;
  background: var(--surface, #ffffff);
  border: 1px solid var(--border, #D6D2D7);
  border-radius: 99px;
  font-size: 0.78rem;
  color: var(--muted, #706273);
}

.fh-default .fh-icon { font-size: 0.72rem; }

/* ── Active: top-right badge ── */
.fh-active {
  top: 1.5rem;
  right: 2rem;
  padding: 0.38rem 1rem 0.38rem 0.8rem;
  background: #321E37;
  border: none;
  border-radius: 99px;
  font-family: 'CoFo Sans Semi-Mono', monospace;
  font-size: 0.88rem;
  font-weight: 500;
  letter-spacing: -0.02em;
  line-height: 1.2;
  color: #50FFEB;
}

.fh-active .fh-icon { font-size: 0.82rem; }
</style>
