<script setup lang="ts">
import {
  ref,
  type Ref,
  useTemplateRef,
  onMounted,
  onUnmounted,
  computed,
} from "vue";

// タイマーの初期値を5分に設定
const DEFAULT_TIMER_MILLISECONDS: number = 1000 * 60 * 5;

const initialTimerMilliseconds: Ref<number> = ref(DEFAULT_TIMER_MILLISECONDS);
const timerMilliseconds: Ref<number> = ref(initialTimerMilliseconds.value);
const totalTimerMilliseconds: Ref<number> = ref(0);
const intervalId: Ref<number | null> = ref(null);
const isTimerActive: Ref<boolean> = ref(false);

onUnmounted(() => {
  clearTimerInterval();
});

const timer: Ref<string> = computed((): string => {
  const totalSeconds = Math.ceil(timerMilliseconds.value / 1000);
  const hours = Math.floor(totalSeconds / 3600);
  const minutes = Math.floor((totalSeconds % 3600) / 60);
  const seconds = totalSeconds % 60;
  if (hours > 0) {
    return `${String(hours)}:${String(minutes).padStart(2, "0")}:${String(
      seconds
    ).padStart(2, "0")}`;
  } else {
    return `${String(minutes)}:${String(seconds).padStart(2, "0")}`;
  }
});

const isTimerRunning: Ref<boolean> = computed((): boolean => {
  return intervalId.value !== null;
});

const timerPercent: Ref<number> = computed((): number => {
  return Math.min(
    Math.max((timerMilliseconds.value / totalTimerMilliseconds.value) * 100, 0),
    100
  );
});

function clearTimerInterval(): void {
  clearInterval(intervalId.value as number);
  intervalId.value = null;
}

function addSeconds(second: number): void {
  timerMilliseconds.value = Math.max(
    timerMilliseconds.value + second * 1000,
    0
  );
}

function startTimer(): void {
  if (!isTimerActive.value) {
    initialTimerMilliseconds.value = timerMilliseconds.value;
    isTimerActive.value = true;
  }
  if (timerMilliseconds.value >= initialTimerMilliseconds.value) {
    totalTimerMilliseconds.value = timerMilliseconds.value;
  }

  intervalId.value = setInterval(() => {
    if (timerMilliseconds.value <= 0) {
      clearTimerInterval();
      return;
    }
    timerMilliseconds.value -= 100;
  }, 100);
}

function resetTimer(): void {
  clearTimerInterval();
  isTimerActive.value = false;
  timerMilliseconds.value = initialTimerMilliseconds.value;
  totalTimerMilliseconds.value = 0;
}

function timerToZero(): void {
  timerMilliseconds.value = 0;
  totalTimerMilliseconds.value = 0;
}
</script>

<template>
  <div class="relative">
    <div ref="timers">
      <div ref="timer-content" class="flex flex-col items-center">
        <div class="relative size-dvw sm:size-[650px] md:size-[750px]">
          <!-- SVG進捗バー -->
          <svg class="absolute inset-0 size-full" viewBox="0 0 100 100">
            <!-- 背景の円 -->
            <circle
              cx="50"
              cy="50"
              r="45"
              fill="none"
              stroke="#e5e7eb"
              stroke-width="3"
              opacity="0.2"
            />
            <!-- 進捗バー -->
            <circle
              class="progress-bar"
              cx="50"
              cy="50"
              r="45"
              fill="none"
              stroke="#00ccff"
              stroke-width="3"
              stroke-linecap="round"
              :stroke-dasharray="90 * Math.PI"
              :stroke-dashoffset="
                90 * Math.PI - (90 * Math.PI * timerPercent) / 100
              "
              transform="rotate(-90 50 50)"
            />
          </svg>
          <!-- 円の中央に重ねるコンテンツ -->
          <div
            class="absolute inset-[15%] flex flex-col items-center justify-center"
          >
            <div class="relative">
              <div class="absolute -top-0 -right-8 sm:-right-12 md:-right-15">
                <button
                  v-if="timerMilliseconds > 0 && !isTimerRunning"
                  class="focusable rounded-sm"
                  @click="timerToZero"
                >
                  <ion-icon
                    class="block w-6 h-6 sm:w-12 sm:h-12 md:w-15 md:h-15"
                    name="close"
                  ></ion-icon>
                </button>
              </div>
              <p
                class="font-[Roboto] font-semibold text-5xl sm:text-8xl md:text-9xl text-center"
              >
                {{ timer }}
              </p>
            </div>
            <div v-if="!isTimerRunning" class="flex gap-2 h-10 my-2">
              <button
                class="focusable w-18 rounded-2xl bg-(--color-background-soft) p-2"
                @click="addSeconds(60 * 10)"
              >
                +10:00
              </button>
              <button
                class="focusable w-18 rounded-2xl bg-(--color-background-soft) p-2"
                @click="addSeconds(60)"
              >
                +1:00
              </button>
              <button
                class="focusable w-18 rounded-2xl bg-(--color-background-soft) p-2"
                @click="addSeconds(10)"
              >
                +0:10
              </button>
            </div>
            <div v-else class="h-10 my-2"></div>
            <div class="flex gap-2 mt-2 w-full">
              <button
                class="grow focusable rounded-2xl p-3 flex flex-col items-center bg-(--color-background-soft)"
                :disabled="timerMilliseconds <= 0"
                @click="isTimerRunning ? clearTimerInterval() : startTimer()"
              >
                <ion-icon
                  class="w-6 h-6"
                  :name="isTimerRunning ? 'pause' : 'play'"
                ></ion-icon>
              </button>
              <button
                v-if="isTimerActive"
                class="grow focusable rounded-2xl p-3 flex flex-col items-center bg-(--color-background-soft)"
                @click="resetTimer"
              >
                <ion-icon class="w-6 h-6" name="refresh"></ion-icon>
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style lang="scss">
.progress-bar {
  transition: stroke-dashoffset 0.3s linear;
}
</style>
