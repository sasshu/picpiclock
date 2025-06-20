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

let initialTimerMilliseconds: number = DEFAULT_TIMER_MILLISECONDS;
const timerMilliseconds: Ref<number> = ref(initialTimerMilliseconds);
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
    initialTimerMilliseconds = timerMilliseconds.value;
    isTimerActive.value = true;
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
  timerMilliseconds.value = initialTimerMilliseconds;
}
</script>

<template>
  <div class="relative">
    <div ref="timers">
      <div ref="timer-content" class="flex flex-col items-center">
        <div class="relative">
          <div class="absolute -top-0 -right-8 sm:-right-12 md:-right-15">
            <button
              v-if="timerMilliseconds > 0 && !isTimerRunning"
              class="focusable rounded-sm"
              @click="() => (timerMilliseconds = 0)"
            >
              <ion-icon
                class="block w-6 h-6 sm:w-12 sm:h-12 md:w-15 md:h-15"
                name="close"
              ></ion-icon>
            </button>
          </div>
          <p
            class="font-[Roboto] font-semibold text-5xl sm:text-9xl md:text-[165px]"
          >
            {{ timer }}
          </p>
        </div>
        <div v-if="!isTimerRunning" class="flex gap-2 h-10 my-4">
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
            class="focusable w-18 rounded-2xl p-2 bg-(--color-background-soft)"
            @click="addSeconds(10)"
          >
            +0:10
          </button>
        </div>
        <div v-else class="h-10 my-4"></div>
        <div class="flex gap-2 mt-2 w-100">
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
</template>

<style lang="scss"></style>
