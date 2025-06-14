<script setup lang="ts">
import { ref, type Ref, useTemplateRef, onMounted, onBeforeUnmount } from "vue";
import { RouterLink, RouterView, useRouter, type Router } from "vue-router";
import HelloWorld from "./components/HelloWorld.vue";

const router: Router = useRouter();
const isMenuOpen: Ref<boolean> = ref(false);
const menuClose: Ref<HTMLElement | null> = useTemplateRef("menu-close");

router.afterEach(() => {
  isMenuOpen.value = false;
});

function toggleMenu() {
  isMenuOpen.value = !isMenuOpen.value;
}
</script>

<template>
  <header>
    <button class="absolute focusable top-[1rem]" @click="toggleMenu">
      <ion-icon class="block w-12 h-12 md:w-15 md:h-15" name="menu"></ion-icon>
    </button>
    <div v-if="isMenuOpen" class="fixed inset-0 z-20">
      <!-- 透明オーバーレイ部分（クリックで閉じる） -->
      <div
        class="absolute inset-0 bg-(--color-background-overlay) bg-opacity-40"
        @click="toggleMenu"
      ></div>
      <!-- メニュー本体 -->
      <nav
        ref="menu-close"
        class="absolute left-0 top-0 h-full w-64 bg-(--color-background) shadow-lg p-4 flex flex-col"
      >
        <button class="focusable mb-4 flex self-start" @click="toggleMenu">
          <ion-icon
            class="block w-12 h-12 md:w-15 md:h-15"
            name="close"
          ></ion-icon>
        </button>
        <ul class="p-[1rem] flex flex-col gap-2">
          <RouterLink
            class="focusable hover:bg-(--color-background-soft) px-2 py-1"
            to="/"
            >時計</RouterLink
          >
          <RouterLink
            class="focusable hover:bg-(--color-background-soft) px-2 py-1"
            to="/about"
            >カウントダウンタイマー</RouterLink
          >
          <RouterLink
            class="focusable hover:bg-(--color-background-soft) px-2 py-1"
            to="/"
            >カウントアップタイマー</RouterLink
          >
        </ul>
      </nav>
    </div>
  </header>

  <div class="flex flex-col items-center justify-center h-screen">
    <RouterView />
  </div>
</template>

<style scoped lang="scss">
.focusable {
  outline: none;
  border-radius: 0.25rem;
  &:focus-visible {
    outline: 2px solid #3b82f6;
  }
}
</style>
