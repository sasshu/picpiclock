<script setup lang="ts">
import { ref, type Ref, useTemplateRef, onMounted, onUnmounted } from "vue";

const currentClock: Ref<ReturnType<typeof getCurrentClock>> = ref(
  getCurrentClock()
);
const intervalId: Ref<number | null> = ref(null);
const pipWindow: Ref<Window | null> = ref(null);
const clock: Ref<HTMLElement | null> = useTemplateRef("clock");
const clockContent: Ref<HTMLElement | null> = useTemplateRef("clock-content");
const isPicDisplay: Ref<boolean> = ref(false);

onMounted(() => {
  initializeClock();
});

onUnmounted(() => {
  clearInterval(intervalId.value as number);
  intervalId.value = null;
});

// 時計を初期化
function initializeClock(): void {
  setTimeout(() => {
    intervalId.value = setInterval(() => {
      currentClock.value = getCurrentClock();
    }, 1000);
  }, 1000 - new Date().getUTCMilliseconds());
}

// 現在の時刻を取得
function getCurrentClock() {
  const date = new Date();
  const hour = date.getHours().toString().padStart(2, "0");
  const minute = date.getMinutes().toString().padStart(2, "0");
  const second = date.getSeconds().toString().padStart(2, "0");

  return {
    time: `${hour}:${minute}:${second}`,
    hour: date.getHours(),
    minute: date.getMinutes(),
    second: date.getSeconds(),
  };
}

// ピクチャーインピクチャー（PIP）を開く
async function openPictureInPicture(): Promise<void> {
  if (
    !("documentPictureInPicture" in window && document.pictureInPictureEnabled)
  ) {
    console.log("Picture-in-Picture is not supported");
    return;
  }
  pipWindow.value = await (
    window.documentPictureInPicture as DocumentPictureInPicture
  ).requestWindow({
    width: 240,
    height: 90,
  });

  // PIPウィンドウにコンテンツを追加
  pipWindow.value.document.body.appendChild(clockContent.value as HTMLElement);

  // PIPウィンドウにCSSを適用
  Array.from(document.styleSheets).forEach((styleSheet) => {
    try {
      const cssRules = Array.from(styleSheet.cssRules)
        .map((rule) => rule.cssText)
        .join("");
      const style = document.createElement("style");

      style.textContent = cssRules;
      pipWindow.value?.document.head.appendChild(style);
    } catch (_) {
      const link = document.createElement("link");
      if (styleSheet.href == null) {
        return;
      }

      link.rel = "stylesheet";
      link.type = styleSheet.type;
      link.media = styleSheet.media.toString();
      link.href = styleSheet.href;
      pipWindow.value?.document.head.appendChild(link);
    }
  });
  isPicDisplay.value = true;

  // PIPウィンドウが閉じられた際に、コンテンツを元の位置に戻す
  pipWindow.value?.addEventListener("pagehide", async () => {
    clock.value?.appendChild(clockContent.value as HTMLElement);
    isPicDisplay.value = false;
  });
}

// PIPウィンドウを閉じる
function closePictureInPicture(): void {
  if (pipWindow.value) {
    pipWindow.value?.close();
    pipWindow.value = null;
    isPicDisplay.value = false;
  }
}
</script>

<template>
  <div ref="clock" class="relative">
    <div
      v-show="!isPicDisplay"
      class="absolute -bottom-8 -right-8 sm:-bottom-12 sm:-right-12 md:-bottom-15 md:-right-15"
    >
      <button class="focusable rounded-sm" @click="openPictureInPicture">
        <ion-icon
          class="block w-6 h-6 sm:w-12 sm:h-12 md:w-15 md:h-15 rotate-90"
          name="open"
        ></ion-icon>
      </button>
    </div>
    <div ref="clock-content">
      <p
        class="font-[Roboto] font-semibold text-5xl sm:text-9xl md:text-[165px]"
      >
        {{ currentClock.time }}
      </p>
    </div>
  </div>
  <div v-if="isPicDisplay" class="text-center">
    <p class="p-3">ピクチャーインピクチャーで表示中です。</p>
    <button
      class="focusable rounded-sm bg-(--color-background-soft) p-2"
      @click="closePictureInPicture"
    >
      表示を戻す
    </button>
  </div>
</template>

<style scoped lang="scss"></style>
