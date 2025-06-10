<template>
  <div class="max-w-5xl mx-auto mt-10 p-6 bg-white shadow-xl rounded-xl flex gap-8">
    <!-- Sidebar Menu -->
    <aside class="w-44 space-y-3">
      <h2 class="text-lg font-semibold text-blue-600">📂 Filter Tugas</h2>
      <button
        v-for="opt in ['all', 'done', 'todo']"
        :key="opt"
        @click="filter = opt"
        :class="[
          'w-full text-left px-4 py-2 rounded-lg transition',
          filter === opt
            ? 'bg-blue-600 text-white shadow'
            : 'bg-gray-100 text-gray-700 hover:bg-gray-200'
        ]"
      >
        {{ filterIcons[opt] }} {{ labels[opt] }}
      </button>
    </aside>

    <!-- Main Content -->
    <main class="flex-1">
      <h1 class="text-3xl font-bold text-blue-700 mb-6 text-center">📝 To-Do List</h1>

      <!-- Form Input -->
      <form @submit.prevent="addTask" class="flex gap-3 mb-4">
        <input
          v-model="newTask"
          :placeholder="placeholder"
          class="flex-1 border p-2 rounded-lg focus:ring-2 focus:ring-blue-500 shadow-sm"
          :class="newTask.length > 40 ? 'border-red-500' : ''"
          :disabled="isLoading"
        />
        <button
          class="bg-blue-600 hover:bg-blue-700 text-white px-4 py-2 rounded-lg font-semibold transition disabled:opacity-50"
          :disabled="!newTask.trim() || newTask.length > 40"
        >
          ➕ Tambah
        </button>
      </form>

      <p v-if="newTask.length > 40" class="text-red-500 text-sm mb-2">
        Maksimal 40 karakter diperbolehkan.
      </p>

      <!-- Task List -->
      <transition-group name="fade" tag="ul" class="space-y-2">
        <li
          v-for="task in filteredTasks"
          :key="task.id"
          class="flex items-center justify-between p-3 bg-gray-50 hover:bg-gray-100 border rounded-lg shadow-sm transition"
        >
          <div
            @click="toggleTask(task)"
            class="cursor-pointer select-none transition"
            :class="{
              'line-through text-gray-400': task.done,
              'text-gray-800': !task.done
            }"
          >
            {{ task.text }}
          </div>
          <button
            @click="deleteTask(task.id)"
            class="text-red-500 hover:text-red-700 text-lg"
            title="Hapus"
          >
            &times;
          </button>
        </li>
      </transition-group>

      <p v-if="filteredTasks.length === 0" class="text-gray-500 text-center mt-6 italic">
        📭 Tidak ada tugas yang sesuai.
      </p>
    </main>
  </div>
</template>

<script setup>
import { ref, watch, onMounted, computed } from 'vue'

const tasks = ref([])
const newTask = ref('')
const isLoading = ref(false)
const filter = ref('all')
const placeholder = ref('Tulis tugas baru...')

const labels = {
  all: 'Semua',
  done: 'Selesai',
  todo: 'Belum Selesai'
}

const filterIcons = {
  all: '📋',
  done: '✅',
  todo: '⏳'
}

onMounted(() => {
  const stored = localStorage.getItem('tasks')
  if (stored) tasks.value = JSON.parse(stored)
})

watch(tasks, (newVal) => {
  localStorage.setItem('tasks', JSON.stringify(newVal))
}, { deep: true })

const addTask = () => {
  if (!newTask.value.trim() || newTask.value.length > 40) return
  tasks.value.push({
    id: Date.now(),
    text: newTask.value.trim(),
    done: false
  })
  newTask.value = ''
}

const toggleTask = (task) => {
  task.done = !task.done
}

const deleteTask = (id) => {
  tasks.value = tasks.value.filter(t => t.id !== id)
}

const filteredTasks = computed(() => {
  if (filter.value === 'done') return tasks.value.filter(t => t.done)
  if (filter.value === 'todo') return tasks.value.filter(t => !t.done)
  return tasks.value
})
</script>

<style>
body {
  font-family: 'Segoe UI', sans-serif;
  background-color: #84a1db;
  margin: 0;
  padding: 0;
}

.fade-enter-active, .fade-leave-active {
  transition: all 0.3s ease;
}
.fade-enter-from, .fade-leave-to {
  opacity: 0;
  transform: translateY(5px);
}
</style>
