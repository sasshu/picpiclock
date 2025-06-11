<script setup lang="ts">
import { ref, type Ref, useTemplateRef, onMounted } from "vue";
import { RouterLink, RouterView } from "vue-router";

const currentClock: Ref<ReturnType<typeof getCurrentClock>> = ref(
  getCurrentClock()
);
const container: Ref<HTMLElement | null> = useTemplateRef("container");
const clockElement: Ref<HTMLElement | null> = useTemplateRef("clock");

onMounted(() => {
  initializeClock();
});

// 時計を初期化
const initializeClock = () => {
  setTimeout(() => {
    const newId = setInterval(() => {
      currentClock.value = getCurrentClock();
    }, 1000);
  }, 1000 - new Date().getUTCMilliseconds());
};

// 現在の時刻を取得
function getCurrentClock() {
  const date = new Date();
  const hour = date.getHours().toString().padStart(2, "0");
  const minute = date.getMinutes().toString().padStart(2, "0");
  const second = date.getSeconds().toString().padStart(2, "0");

  return {
    time: `${hour}:${minute} ${second}`,
    hour: date.getHours(),
    minute: date.getMinutes(),
    second: date.getSeconds(),
  };
}

// ピクチャーインピクチャー（PIP）を開く
async function openPictureInPicture() {
  if (!document.pictureInPictureEnabled) {
    console.log("Picture-in-Picture is not supported");
    return;
  }
  const pipWindow: Window = await window.documentPictureInPicture.requestWindow(
    {
      width: 400,
      height: 300,
    }
  );
  // 5. ピクチャーインピクチャーの背景色を設定
  //   pipBackground = window.getComputedStyle(pipContent).backgroundColor;
  //   pipWindow.document.body.style.backgroundColor = pipBackground;
  //   pipWindow.document.body.style.margin = "0";

  pipWindow.addEventListener("load", async () => {
    pipWindow.document.adoptedStyleSheets = [...document.styleSheets];
    // pipWindow.document.body.append(clockElement.value as HTMLElement);
  });
  pipWindow.document.body.append(clockElement.value as HTMLElement);

  // PIPウィンドウが閉じられた際に、コンテンツを元の位置に戻す
  pipWindow.addEventListener("pagehide", () => {
    container.value?.append(clockElement.value as HTMLElement);
  });
}
</script>

<template>
  <div ref="container">
    <button class="text-6xl" @click="openPictureInPicture">
      <ion-icon name="open"></ion-icon>
    </button>
    <p ref="clock" class="font-[Roboto] font-semibold text-[150px]">
      {{ currentClock.time }}
    </p>
  </div>
</template>

<style scoped lang="scss"></style>
