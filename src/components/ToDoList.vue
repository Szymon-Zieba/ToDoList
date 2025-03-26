<script setup lang="ts">
import AddButton from "./AddButton.vue";
import { ref, onMounted, computed, watch } from "vue";
import Button from "./Button.vue";
import { CheckIcon, MinusCircleIcon, PencilSquareIcon, MagnifyingGlassIcon, AdjustmentsHorizontalIcon, ArrowsUpDownIcon } from "@heroicons/vue/24/solid";
const tasks = ref<{ title: string; description: string; status: string, date: string, id: number, isDone?: boolean }[]>([]);
const isFormOpen = ref(false);
const closeForm = () => isFormOpen.value = false;
const isFormEditOpen = ref(false);
const closeFormEdit = () => isFormEditOpen.value = false;
const currentTaskIndex = ref<number | null>(null);

const title = ref("");
const description = ref("");
const status = ref("To Do");
const isSubmitted = ref(false);
const errors = ref<{ title?: string; description?: string; status?: string }>({});
const searchQuery = ref("");
const sortOrder = ref("All");
const sortDateOrder = ref("desc");
const debouncedSearchQuery = ref("");
const debounceTimeout = ref<ReturnType<typeof setTimeout> | null>(null);

watch(searchQuery, (newValue) => {
  if (debounceTimeout.value) clearTimeout(debounceTimeout.value);
  debounceTimeout.value = setTimeout(() => {
    debouncedSearchQuery.value = newValue;
  }, 300);
});

onMounted(() => {
  const savedTasks = localStorage.getItem("tasks");
  if (savedTasks) {
    tasks.value = JSON.parse(savedTasks);
  }
});

watch(tasks, (newTasks) => {
  localStorage.setItem("tasks", JSON.stringify(newTasks));
}, { deep: true });

const validateForm = () => {
  errors.value = {};
  if (title.value.length < 3 || title.value.length > 50) {
    errors.value.title = "Title must be at least 3 characters (max 50 characters).";
  }
  if (description.value.length === 0 || description.value.length > 200) {
    errors.value.description = "Description is required (max 200 characters).";
  }
  if (!status.value) {
    errors.value.status = "Please select a status.";
  }
  return Object.keys(errors.value).length === 0;
};

const handleIsFormOpen = () => {
  closeFormEdit();
  isFormOpen.value = !isFormOpen.value;
  isSubmitted.value = false;
  if (!isFormOpen.value) resetForm();
};

const resetForm = () => {
  title.value = "";
  description.value = "";
  status.value = "To Do";
  errors.value = {};
  currentTaskIndex.value = null;
};

const addTask = (id: number | Event) => {
  isSubmitted.value = true;
  if (!validateForm()) return;

  const newTask = {
    id: Date.now(),
    title: title.value,
    description: description.value,
    status: status.value,
    isDone: false,
    date: new Date().toISOString(),
  };

  if (typeof id === "number") {
    tasks.value = tasks.value.map(task =>
        task.id === id ? newTask : task
    );
  } else {
    tasks.value.push(newTask);
  }

  closeForm();
  closeFormEdit();
  resetForm();
};

const removeTask = (taskId: number) => {
  tasks.value = tasks.value.filter(task => task.id !== taskId);
};

const filteredTasks = computed(() => {
  let filtered = tasks.value.filter(task =>
      debouncedSearchQuery.value === "" ||
      task.title.toLowerCase().includes(debouncedSearchQuery.value.toLowerCase()) ||
      task.description.toLowerCase().includes(debouncedSearchQuery.value.toLowerCase())
  );

  if (sortOrder.value !== "All") {
    filtered = filtered.filter(task => task.status === sortOrder.value);
  }

  return filtered.sort((a, b) => {
    return sortDateOrder.value === "asc"
        ? new Date(a.date).getTime() - new Date(b.date).getTime()
        : new Date(b.date).getTime() - new Date(a.date).getTime();
  });
});

const editTask = (taskId: number, index: number) => {
  const task = tasks.value.find(task => task.id === taskId);
  if (task) {
    currentTaskIndex.value = index;
    title.value = task.title;
    description.value = task.description;
    status.value = task.status;
    isFormOpen.value = false;
    isFormEditOpen.value = true;
  }
};

const swapFinishTask = (taskId: number) => {
  const task = tasks.value.find(task => task.id === taskId);
  if (task) {
    task.isDone = !task.isDone;
  }
};
</script>

<template>
  <div class="mx-auto max-w-4xl p-4">
    <h1 class="text-3xl font-bold text-center mt-8">To Do List</h1>

    <div class="mt-4 flex flex-col md:flex-row md:items-center md:space-x-4 space-y-4 md:space-y-0">

      <div class="relative w-full md:w-2/3">
        <MagnifyingGlassIcon class="absolute left-3 top-3 w-5 h-5 text-gray-400" />
        <input
          v-model="searchQuery"
          type="text"
          class="w-full border border-gray-300 rounded-lg pl-10 pr-4 py-2 focus:ring-2 focus:ring-blue-400 focus:outline-none"
        />
      </div>

      <div class="relative w-full md:w-1/3">
        <AdjustmentsHorizontalIcon class="absolute left-3 top-3 w-5 h-5 text-gray-400" />
        <select
          v-model="sortOrder"
          class="w-full border border-gray-300 rounded-lg pl-10 pr-4 py-2 focus:ring-2 focus:ring-blue-400 focus:outline-none"
        >
          <option value="All">All</option>
          <option value="To Do">To Do</option>
          <option value="In Progress">In Progress</option>
          <option value="Done">Done</option>
        </select>
      </div>

      <div class="relative w-full md:w-1/3">
        <ArrowsUpDownIcon class="absolute left-3 top-3 w-5 h-5 text-gray-400" />
        <select
          v-model="sortDateOrder"
          class="w-full border border-gray-300 rounded-lg pl-10 pr-4 py-2 focus:ring-2 focus:ring-blue-400 focus:outline-none"
        >
          <option value="desc">Desc</option>
          <option value="asc">Asc</option>
        </select>
      </div>

    </div>

    <ul class="mt-8">
      <div v-show="filteredTasks.length === 0">
        No tasks found.
      </div>
      <li
        v-for="(task, index) in filteredTasks"
        :key="index"
      >
        <div
          class="p-4 rounded-lg mb-4 border border-gray-400 flex justify-between items-center gap-4"
          :class="{'bg-stone-500 opacity-90': task.isDone }"
        >
          <div
            class="flex gap-4 items-center"
          >
            <Button
              class="min-w-7 min-h-7 border border-gray-400 rounded-full group flex items-center justify-center p-0.5"
              @click="swapFinishTask(task.id)"
            >
              <CheckIcon
                class="text-inherit pl-0.5"
                :class="{
                  'group-hover:hidden': task.isDone,
                  'hidden group-hover:block': !task.isDone
                }"
              />
            </Button>
            <div class="w-full">
              <span
                class="text-sm font-medium text-white px-2 py-1 rounded"
                :class="{
                  'bg-blue-500': task.status === 'To Do',
                  'bg-yellow-500': task.status === 'In Progress',
                  'bg-green-500': task.status === 'Done'
                }"
                >
                  {{ task.status }}
                </span>
              <h2 class="font-semibold text-lg mt-2 break-all">{{ task.title }}</h2>
              <span class="text-gray-400 break-all">{{ task.description }}</span>
              <p class="text-gray-400 text-sm mt-1">{{ new Date(task.date).toLocaleString() }}</p>
            </div>
          </div>
          <div class="flex gap-4">
            <Button @click="editTask(task.id, index)"><PencilSquareIcon class="w-8 h-8" /></Button>
            <Button @click="removeTask(task.id)"><MinusCircleIcon class="w-8 h-8" /></Button>
          </div>
        </div>

        <div v-show="currentTaskIndex === index && isFormEditOpen" class="border border-gray-400 rounded-lg p-4 mb-4">
          <input
              placeholder="Wpisz tytuł zadania"
              v-model="title"
              type="text"
              class="w-full text-red-400 placeholder-gray-500 text-lg"
              :class="{ 'border-red-500': isSubmitted && errors.title }"
          />
          <p v-if="isSubmitted && errors.title" class="text-red-500 text-sm">{{ errors.title }}</p>

          <br/>

          <textarea
              placeholder="Opis"
              v-model="description"
              class="w-full resize-none text-gray-400 placeholder-gray-500"
              rows="3"
              maxlength="200"
              :class="{ 'border-red-500': isSubmitted && errors.description }"
          />
          <p v-if="isSubmitted && errors.description" class="text-red-500 text-sm">{{ errors.description }}</p>

          <div class="md:w-1/4">
            <label class="block font-semibold mt-3">Status</label>
            <select
                v-model="status"
                class="w-full border border-gray-300 rounded-lg pl-3 pr-4 py-2 focus:ring-2 focus:ring-blue-400 focus:outline-none"
                :class="{ 'border-red-500': isSubmitted && errors.status }"
            >
              <option value="To Do">To Do</option>
              <option value="In Progress">In Progress</option>
              <option value="Done">Done</option>
            </select>
            <p v-if="isSubmitted && errors.status" class="text-red-500 text-sm">{{ errors.status }}</p>
          </div>

          <div class="flex justify-end gap-4 mt-4">
            <Button @click="closeFormEdit">Cancel</Button>
            <Button @click="addTask(task.id)">Save</Button>
          </div>
        </div>
      </li>
    </ul>

    <div v-if="isFormOpen" class="border border-gray-400 rounded-lg p-4">
      <input
          placeholder="Wpisz tytuł zadania"
          v-model="title"
          type="text"
          class="w-full text-red-400 placeholder-gray-500 text-lg"
          :class="{ 'border-red-500': isSubmitted && errors.title }"
      />
      <p v-if="isSubmitted && errors.title" class="text-red-500 text-sm">{{ errors.title }}</p>

      <br/>

      <textarea
          placeholder="Opis"
          v-model="description"
          class="w-full resize-none text-gray-400 placeholder-gray-500"
          rows="3"
          maxlength="200"
          :class="{ 'border-red-500': isSubmitted && errors.description }"
      />
      <p v-if="isSubmitted && errors.description" class="text-red-500 text-sm">{{ errors.description }}</p>

      <div class="md:w-1/4">
        <label class="block font-semibold mt-3">Status</label>
        <select
            v-model="status"
            class="w-full border border-gray-300 rounded-lg pl-3 pr-4 py-2 focus:ring-2 focus:ring-blue-400 focus:outline-none"
            :class="{ 'border-red-500': isSubmitted && errors.status }"
        >
          <option value="To Do">To Do</option>
          <option value="In Progress">In Progress</option>
          <option value="Done">Done</option>
        </select>
        <p v-if="isSubmitted && errors.status" class="text-red-500 text-sm">{{ errors.status }}</p>
      </div>

      <div class="flex justify-end gap-4 mt-4">
        <Button @click="closeForm">Cancel</Button>
        <Button @click="addTask">Save</Button>
      </div>
    </div>
    <AddButton class="mt-10" @click="handleIsFormOpen" v-else />
  </div>
</template>
