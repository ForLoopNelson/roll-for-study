<template>
  <div class="frame">
    <h1>Roll to Study</h1>

    <div class="die-wrap">
      <div class="die" :class="{ rolling: isRolling }" @click="roll">{{ dieFace }}</div>
    </div>
    <button class="roll-btn" @click="roll" :disabled="isRolling">
      {{ hasRolled ? 'Roll Again' : 'Roll the d20' }}
    </button>

    <div class="result" v-if="current">
      <div class="task-name">{{ current.name }}</div>
      <div class="task-sub">Rolled {{ lastRoll }} &middot; {{ current.minutes }} min</div>

      <div class="done-msg" v-if="finished">Time's up — nice work.</div>
      <div class="timer">{{ display }}</div>

      <div class="controls">
        <button class="primary" @click="toggle" v-if="!finished">
          {{ running ? 'Pause' : (secondsLeft === current.minutes * 60 ? 'Start' : 'Resume') }}
        </button>
        <button @click="reset">Reset</button>
      </div>
    </div>
    <div class="placeholder" v-else>Roll to get a task and a timer.</div>
  </div>
</template>

<script setup>
import { ref, computed, onUnmounted } from 'vue';

// Edit this list to change outcomes, ranges, or durations.
const TASKS = [
  { range: [1],   name: 'Deep Study',  minutes: 25 },
  { range: [2,6],   name: 'Quick Study',  minutes: 10 },
  { range: [7, 10],  name: 'Quick Break', minutes: 5 },
  { range: [11, 14], name: 'Chore',       minutes: 15 },
  { range: [15, 17], name: 'Read',    minutes: 10 },
  { range: [18, 19], name: 'Exercise',    minutes: 15 },
  { range: [20], name: 'Treat Yourself',  minutes: 30 },
];

function taskForRoll(n) {
  return TASKS.find(t => n >= t.range[0] && n <= t.range[1]) || TASKS[0];
}

const dieFace = ref('20');
const isRolling = ref(false);
const hasRolled = ref(false);
const lastRoll = ref(null);
const current = ref(null);
const secondsLeft = ref(0);
const running = ref(false);
const finished = ref(false);
let timerId = null;

function clearTimer() {
  if (timerId) {
    clearInterval(timerId);
    timerId = null;
  }
}

function roll() {
  if (isRolling.value) return;
  clearTimer();
  running.value = false;
  finished.value = false;
  isRolling.value = true;

  let ticks = 0;
  const spin = setInterval(() => {
    dieFace.value = String(1 + Math.floor(Math.random() * 20));
    ticks++;
    if (ticks > 8) {
      clearInterval(spin);
      const n = 1 + Math.floor(Math.random() * 20);
      dieFace.value = String(n);
      lastRoll.value = n;
      current.value = taskForRoll(n);
      secondsLeft.value = current.value.minutes * 60;
      hasRolled.value = true;
      isRolling.value = false;
    }
  }, 55);
}

function tick() {
  if (secondsLeft.value <= 0) {
    running.value = false;
    finished.value = true;
    clearTimer();
    return;
  }
  secondsLeft.value--;
  if (secondsLeft.value === 0) {
    running.value = false;
    finished.value = true;
    clearTimer();
  }
}

function toggle() {
  if (!current.value) return;
  if (running.value) {
    running.value = false;
    clearTimer();
  } else {
    running.value = true;
    finished.value = false;
    clearTimer();
    timerId = setInterval(tick, 1000);
  }
}

function reset() {
  clearTimer();
  running.value = false;
  finished.value = false;
  if (current.value) secondsLeft.value = current.value.minutes * 60;
}

const display = computed(() => {
  const m = Math.floor(secondsLeft.value / 60).toString().padStart(2, '0');
  const s = (secondsLeft.value % 60).toString().padStart(2, '0');
  return `${m}:${s}`;
});

onUnmounted(clearTimer);
</script>

<style scoped>

.frame {
  background: var(--beige-dark);
  border: 2px solid var(--black);
  border-radius: 16px;
  padding: 28px 24px;
  text-align: center;
  box-shadow: 0 8px 0 var(--black);
  max-width: 420px;
  margin: 0 auto;
  font: 16px/1.4 system-ui, 'Segoe UI', Roboto, sans-serif;
  color: var(--ink);
}
h1 {
  margin: 0 0 20px;
  font-size: 22px;
  letter-spacing: 0.5px;
  color: var(--black);
}
.die-wrap {
  display: flex;
  justify-content: center;
  margin-bottom: 18px;
}
.die {
  width: 110px;
  height: 110px;
  background: var(--red);
  border: 3px solid var(--black);
  clip-path: polygon(50% 0%, 100% 25%, 100% 75%, 50% 100%, 0% 75%, 0% 25%);
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  color: var(--beige);
  font-size: 34px;
  font-weight: 700;
  transition: transform 0.12s ease;
  user-select: none;
}
.die:active {
  transform: scale(0.92);
}
.die.rolling {
  animation: spin 0.5s ease;
}
@keyframes spin {
  0% { transform: rotate(0deg) scale(1); }
  50% { transform: rotate(180deg) scale(1.08); }
  100% { transform: rotate(360deg) scale(1); }
}
.roll-btn {
  display: block;
  width: 100%;
  background: var(--black);
  color: var(--beige);
  border: none;
  border-radius: 10px;
  padding: 12px;
  font-size: 15px;
  font-weight: 600;
  cursor: pointer;
  margin-bottom: 20px;
  letter-spacing: 0.3px;
}
.roll-btn:hover {
  background: var(--red-dark);
}
.result {
  background: var(--beige);
  border: 2px solid var(--black);
  border-radius: 12px;
  padding: 18px;
  margin-bottom: 18px;
}
.task-name {
  font-size: 20px;
  font-weight: 700;
  color: var(--red-dark);
  margin-bottom: 4px;
}
.task-sub {
  font-size: 13px;
  color: var(--ink);
  opacity: 0.75;
  margin-bottom: 14px;
}
.timer {
  font-size: 44px;
  font-weight: 700;
  color: var(--black);
  font-variant-numeric: tabular-nums;
  margin-bottom: 12px;
}
.controls {
  display: flex;
  gap: 10px;
  justify-content: center;
}
.controls button {
  flex: 1;
  padding: 10px;
  border-radius: 8px;
  border: 2px solid var(--black);
  background: var(--beige-dark);
  color: var(--black);
  font-weight: 600;
  font-size: 14px;
  cursor: pointer;
}
.controls button.primary {
  background: var(--red);
  color: var(--beige);
  border-color: var(--red-dark);
}
.controls button:hover {
  opacity: 0.85;
}
.placeholder {
  font-size: 14px;
  opacity: 0.7;
  padding: 10px 0;
}
.done-msg {
  font-weight: 700;
  color: var(--red-dark);
  margin-bottom: 10px;
}
</style>