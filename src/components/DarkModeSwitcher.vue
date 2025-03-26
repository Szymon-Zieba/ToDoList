<script setup lang="ts">
import { ref, onMounted } from "vue";
import { MoonIcon, SunIcon } from "@heroicons/vue/24/solid";

const isDarkMode = ref(false);

const toggleDarkMode = () => {
  isDarkMode.value = !isDarkMode.value;
  document.documentElement.classList.toggle("dark", isDarkMode.value);
  localStorage.setItem("theme", isDarkMode.value ? "dark" : "light");
};

onMounted(() => {
  const savedTheme = localStorage.getItem("theme");
  if (savedTheme === "dark") {
    isDarkMode.value = true;
    document.documentElement.classList.add("dark");
  }
});
</script>

<template>
  <button
      @click="toggleDarkMode"
      class="p-2 rounded-full bg-gray-200 dark:bg-gray-800 text-gray-900 dark:text-gray-100 hover:bg-gray-300 dark:hover:bg-gray-700 transition cursor-pointer"
  >
    <MoonIcon v-if="!isDarkMode" class="w-6 h-6" />
    <SunIcon v-else class="w-6 h-6" />
  </button>
</template>
