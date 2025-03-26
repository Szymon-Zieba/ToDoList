<script setup lang="ts">

import Button from "./Button.vue";
import {PlusIcon} from "@heroicons/vue/24/solid";
</script>

<template>
  <Button>
    <PlusIcon class="h-7 w-7 text-red-400 group-hover:text-white group-hover:bg-red-400 rounded-full p-0.5"/>
    Add Task
  </Button>
</template>
