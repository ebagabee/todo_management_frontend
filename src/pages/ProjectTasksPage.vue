<template>
  <q-page padding>
    <h1 class="text-h4 q-mb-md">Tarefas do Projeto: {{ project?.name }}</h1>

    <q-btn color="primary" label="Nova Tarefa" @click="openNewTaskDialog" class="q-mb-md" />

    <div v-if="loading">Carregando tarefas...</div>

    <div v-else-if="tasks.length === 0" class="text-h6 text-center q-pa-md">
      Nenhuma tarefa associada a este projeto
    </div>

    <q-list v-else bordered separator>
      <q-item v-for="task in tasks" :key="task.id" clickable @click="openTaskModal(task)">
        <q-item-section>
          <q-item-label>{{ task.title }}</q-item-label>
          <q-item-label caption>{{ task.description }}</q-item-label>
        </q-item-section>
        <q-item-section side>
          <q-badge :color="getStatusColor(task.status)">{{ translateStatus(task.status) }}</q-badge>
          <q-badge :color="getPriorityColor(task.priority)" class="q-ml-sm q-mt-xs">{{
            translatePriority(task.priority)
          }}</q-badge>
        </q-item-section>
      </q-item>
    </q-list>

    <q-dialog v-model="newTaskDialog" persistent>
      <q-card style="min-width: 350px; max-width: 600px; width: 90vw">
        <q-card-section class="bg-primary text-white">
          <div class="text-h6">Nova Tarefa</div>
          <q-btn flat round dense icon="close" v-close-popup class="absolute-top-right" />
        </q-card-section>

        <q-card-section class="q-pt-md">
          <q-input v-model="newTask.title" label="Título" filled />
          <q-input
            v-model="newTask.description"
            type="textarea"
            label="Descrição"
            filled
            class="q-mt-md"
            autogrow
          />

          <div class="row q-col-gutter-md q-mt-md">
            <div class="col-12 col-sm-6">
              <q-select
                v-model="newTask.status"
                :options="statusOptions"
                option-value="value"
                option-label="label"
                label="Status"
                filled
                emit-value
                map-options
              />
            </div>
            <div class="col-12 col-sm-6">
              <q-select
                v-model="newTask.priority"
                :options="priorityOptions"
                option-value="value"
                option-label="label"
                label="Prioridade"
                filled
                emit-value
                map-options
              />
            </div>
          </div>
        </q-card-section>

        <q-card-actions align="right" class="bg-grey-1">
          <q-btn flat label="Cancelar" color="grey" v-close-popup />
          <q-btn flat label="Criar" color="primary" @click="createTask" />
        </q-card-actions>
      </q-card>
    </q-dialog>

    <q-dialog v-model="taskModalVisible" persistent>
      <q-card style="min-width: 350px; max-width: 600px; width: 90vw">
        <q-card-section class="bg-primary text-white">
          <div class="text-h6">Detalhes da Tarefa</div>
          <q-btn flat round dense icon="close" v-close-popup class="absolute-top-right" />
        </q-card-section>

        <q-card-section v-if="selectedTask" class="q-pt-md">
          <q-input
            v-model="selectedTask.title"
            label="Título"
            filled
            :rules="[(val) => !!val || 'Título é obrigatório']"
          />
          <q-input
            v-model="selectedTask.description"
            type="textarea"
            label="Descrição"
            filled
            class="q-mt-md"
            autogrow
          />

          <div class="row q-col-gutter-md q-mt-md">
            <div class="col-12 col-sm-6">
              <q-select
                v-model="selectedTask.status"
                :options="statusOptions"
                option-value="value"
                option-label="label"
                label="Status"
                filled
                emit-value
                map-options
              />
            </div>
            <div class="col-12 col-sm-6">
              <q-select
                v-model="selectedTask.priority"
                :options="priorityOptions"
                option-value="value"
                option-label="label"
                label="Prioridade"
                filled
                emit-value
                map-options
              />
            </div>
          </div>
        </q-card-section>

        <q-card-actions align="right" class="bg-grey-1">
          <q-btn flat label="Excluir" color="negative" @click="confirmDeleteTask" />
          <q-space />
          <q-btn flat label="Cancelar" color="grey" v-close-popup />
          <q-btn flat label="Salvar" color="primary" @click="updateTask" />
        </q-card-actions>
      </q-card>
    </q-dialog>

    <q-dialog v-model="deleteConfirmationDialog" persistent>
      <q-card>
        <q-card-section class="row items-center">
          <q-avatar icon="warning" color="negative" text-color="white" />
          <span class="q-ml-sm">Tem certeza que deseja excluir esta tarefa?</span>
        </q-card-section>

        <q-card-actions align="right">
          <q-btn flat label="Cancelar" color="primary" v-close-popup />
          <q-btn flat label="Excluir" color="negative" @click="deleteTask" />
        </q-card-actions>
      </q-card>
    </q-dialog>
  </q-page>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { useRoute } from 'vue-router'
import { api } from 'src/boot/axios'

const route = useRoute()
const projectId = Number(route.params.id)

interface Project {
  id: number
  name: string
}

interface Task {
  id: number
  title: string
  description: string
  status: 'pending' | 'in_progress' | 'completed'
  priority: 'low' | 'medium' | 'high' | 'urgent'
  project_id: number
}

type StatusOption = { value: 'pending' | 'in_progress' | 'completed'; label: string }
type PriorityOption = { value: 'low' | 'medium' | 'high' | 'urgent'; label: string }

interface NewTask {
  title: string
  description?: string
  status: 'pending' | 'in_progress' | 'completed'
  priority: 'low' | 'medium' | 'high' | 'urgent'
  project_id: number
}

const project = ref<Project | null>(null)
const tasks = ref<Task[]>([])
const loading = ref(true)
const newTaskDialog = ref(false)
const selectedTask = ref<Task | null>(null)
const taskModalVisible = ref(false)
const deleteConfirmationDialog = ref(false)

const statusOptions: StatusOption[] = [
  { value: 'pending', label: 'Pendente' },
  { value: 'in_progress', label: 'Em Andamento' },
  { value: 'completed', label: 'Concluída' },
]

const priorityOptions: PriorityOption[] = [
  { value: 'low', label: 'Baixa' },
  { value: 'medium', label: 'Média' },
  { value: 'high', label: 'Alta' },
  { value: 'urgent', label: 'Urgente' },
]

const newTask = ref<NewTask>({
  title: '',
  description: '',
  status: 'pending',
  priority: 'medium',
  project_id: projectId,
})

onMounted(async () => {
  try {
    const [projectResponse, tasksResponse] = await Promise.all([
      api.get<Project>(`/projects/${projectId}`),
      api.get<Task[]>(`/projects/${projectId}/tasks`),
    ])
    project.value = projectResponse.data
    tasks.value = tasksResponse.data
  } catch (error) {
    console.error('Error loading project data:', error)
  } finally {
    loading.value = false
  }
})

const openNewTaskDialog = () => {
  newTask.value = {
    title: '',
    description: '',
    status: 'pending',
    priority: 'medium',
    project_id: projectId,
  }
  newTaskDialog.value = true
}

const createTask = async () => {
  try {
    if (!newTask.value.title.trim()) {
      return
    }

    const taskData: NewTask = {
      title: newTask.value.title,
      description: newTask.value.description || '',
      status: newTask.value.status,
      priority: newTask.value.priority,
      project_id: projectId,
    }

    const response = await api.post<Task>('/tasks', taskData)
    tasks.value.push(response.data)
    newTaskDialog.value = false
  } catch (error) {
    console.error('Error creating task:', error)
  }
}

const updateTask = async () => {
  if (!selectedTask.value) return

  try {
    const taskData = {
      title: selectedTask.value.title,
      description: selectedTask.value.description || '',
      status: selectedTask.value.status,
      priority: selectedTask.value.priority,
    }

    const response = await api.put<Task>(`/tasks/${selectedTask.value.id}`, taskData)
    const index = tasks.value.findIndex((t) => t.id === response.data.id)
    if (index !== -1) {
      tasks.value[index] = response.data
    }
    taskModalVisible.value = false
  } catch (error) {
    console.error('Error updating task:', error)
  }
}

const getStatusColor = (status: Task['status']) => {
  const colors: Record<Task['status'], string> = {
    pending: 'orange',
    in_progress: 'blue',
    completed: 'green',
  }
  return colors[status]
}

const translateStatus = (status: Task['status']): string => {
  const translations: Record<Task['status'], string> = {
    pending: 'Pendente',
    in_progress: 'Em Andamento',
    completed: 'Concluída',
  }
  return translations[status]
}

const translatePriority = (priority: Task['priority']): string => {
  const translations: Record<Task['priority'], string> = {
    low: 'Baixa',
    medium: 'Média',
    high: 'Alta',
    urgent: 'Urgente',
  }
  return translations[priority]
}

const getPriorityColor = (priority: Task['priority']) => {
  const colors: Record<Task['priority'], string> = {
    low: 'green',
    medium: 'blue',
    high: 'orange',
    urgent: 'red',
  }
  return colors[priority]
}

const openTaskModal = (task: Task) => {
  selectedTask.value = { ...task }
  taskModalVisible.value = true
}

const confirmDeleteTask = () => {
  deleteConfirmationDialog.value = true
}

const deleteTask = async () => {
  if (!selectedTask.value) return

  try {
    await api.delete(`/tasks/${selectedTask.value.id}`)
    const index = tasks.value.findIndex((t) => t.id === selectedTask.value!.id)
    if (index !== -1) {
      tasks.value.splice(index, 1)
    }
    taskModalVisible.value = false
    deleteConfirmationDialog.value = false
  } catch (error) {
    console.error('Erro ao excluir tarefa:', error)
  }
}
</script>
