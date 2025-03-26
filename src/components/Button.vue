<script setup lang="ts">

defineProps<{
  text?: string;
}>();

</script>

<template>
  <button class="flex items-center gap-2 text-lg hover:text-red-400 group cursor-pointer">
    <slot>{{ text }}</slot>
  </button>
</template>
