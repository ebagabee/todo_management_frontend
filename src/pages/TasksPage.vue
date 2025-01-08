<template>
  <q-page class="q-pa-md">
    <div class="row justify-center q-mb-md">
      <div class="col-12 col-md-8 col-lg-6">
        <h2 class="text-h4 text-weight-bold text-primary q-mb-md">Lista de Tarefas</h2>
        <add-task-form @add-task="addTask" />
      </div>
    </div>

    <div class="row justify-center q-mt-md">
      <div class="col-12 col-md-8 col-lg-6">
        <q-list bordered separator>
          <task-item v-for="task in tasks" :key="task.id" :task="task" @view-task="openTaskModal" />
        </q-list>
      </div>
    </div>

    <task-modal
      v-if="selectedTask"
      v-model="isTaskModalOpen"
      :task="selectedTask"
      @update-task="updateTask"
    />
  </q-page>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { Notify } from 'quasar'
import axios from 'axios'
import AddTaskForm from 'components/AddTaskForm.vue'
import TaskItem from 'components/TaskItem.vue'
import TaskModal from 'components/TaskModal.vue'

interface Task {
  id: number
  title: string
  description: string
  status: 'pending' | 'in_progress' | 'completed'
  created_at: string
}

const tasks = ref<Task[]>([])
const selectedTask = ref<Task | null>(null)
const isTaskModalOpen = ref(false)

const fetchTasks = async () => {
  try {
    const response = await axios.get('http://localhost:8000/api/tasks')
    tasks.value = response.data
  } catch (error) {
    console.error('Erro ao buscar tarefas:', error)
    Notify.create({
      color: 'negative',
      message: 'Erro ao carregar tarefas.',
      icon: 'error',
    })
  }
}

const addTask = async (task: { title: string; description: string; status: string }) => {
  try {
    await axios.post('http://localhost:8000/api/tasks', task)
    await fetchTasks()
    Notify.create({
      color: 'positive',
      message: 'Tarefa adicionada com sucesso!',
      icon: 'check',
    })
  } catch (error) {
    console.error('Erro ao adicionar tarefa:', error)
    Notify.create({
      color: 'negative',
      message: 'Erro ao adicionar tarefa.',
      icon: 'error',
    })
  }
}

const updateTask = async (task: Omit<Task, 'created_at'>) => {
  try {
    await axios.put(`http://localhost:8000/api/tasks/${task.id}`, task)
    await fetchTasks()
    Notify.create({
      color: 'positive',
      message: 'Tarefa atualizada com sucesso!',
      icon: 'update',
    })
  } catch (error) {
    console.error('Erro ao atualizar tarefa:', error)
    Notify.create({
      color: 'negative',
      message: 'Erro ao atualizar tarefa.',
      icon: 'error',
    })
  }
}

const openTaskModal = (task: Task) => {
  selectedTask.value = task
  isTaskModalOpen.value = true
}

onMounted(fetchTasks)
</script>
