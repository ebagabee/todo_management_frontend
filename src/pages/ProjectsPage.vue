<template>
  <q-page padding>
    <h1 class="text-h4 q-mb-md">Projetos</h1>
    <q-btn color="primary" label="Novo Projeto" @click="openNewProjectDialog" />

    <q-list bordered separator class="q-mt-md">
      <q-item
        v-for="project in projects"
        :key="project.id"
        clickable
        @click="goToProjectTasks(project.id)"
      >
        <q-item-section>
          <q-item-label>{{ project.name }}</q-item-label>
          <q-item-label caption>{{ project.description }}</q-item-label>
        </q-item-section>
        <q-item-section side>
          <q-badge :color="getStatusColor(project.status)">{{
            translateStatus(project.status)
          }}</q-badge>
        </q-item-section>
      </q-item>
    </q-list>

    <q-dialog v-model="newProjectDialog">
      <q-card style="min-width: 350px">
        <q-card-section>
          <div class="text-h6">Novo Projeto</div>
        </q-card-section>

        <q-card-section class="q-pt-none">
          <q-input v-model="newProject.name" label="Nome do Projeto" />
          <q-input v-model="newProject.description" type="textarea" label="Descrição" />
          <q-input v-model="newProject.start_date" type="date" label="Data de Início" />
          <q-select v-model="newProject.status" :options="statusOptions" label="Status" />
        </q-card-section>

        <q-card-actions align="right">
          <q-btn flat label="Cancelar" color="primary" v-close-popup />
          <q-btn flat label="Criar" color="primary" @click="createProject" v-close-popup />
        </q-card-actions>
      </q-card>
    </q-dialog>
  </q-page>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { api } from 'src/boot/axios'
import { useRouter } from 'vue-router'

const router = useRouter()

interface Project {
  id: number
  name: string
  description: string
  start_date: string
  status: 'planning' | 'active' | 'on_hold' | 'completed'
}

interface NewProject {
  name: string
  description: string
  start_date: string
  status: 'planning' | 'active' | 'on_hold' | 'completed'
}

const projects = ref<Project[]>([])
const newProjectDialog = ref(false)
const newProject = ref<NewProject>({
  name: '',
  description: '',
  start_date: '',
  status: 'planning',
})

const statusOptions = [
  { value: 'planning', label: 'Planejamento' },
  { value: 'active', label: 'Ativo' },
  { value: 'on_hold', label: 'Em espera' },
  { value: 'completed', label: 'Concluído' },
]

onMounted(async () => {
  try {
    const response = await api.get<Project[]>('/projects')
    projects.value = response.data
  } catch (error) {
    console.error('Error fetching projects:', error)
  }
})

const openNewProjectDialog = () => {
  newProjectDialog.value = true
}

const createProject = async () => {
  try {
    const response = await api.post<Project>('/projects', newProject.value)
    projects.value.push(response.data)
    newProject.value = { name: '', description: '', start_date: '', status: 'planning' }
  } catch (error) {
    console.error('Error creating project:', error)
  }
}

const goToProjectTasks = (projectId: number) => {
  router.push(`/projects/${projectId}/tasks`)
}

const getStatusColor = (status: Project['status']) => {
  const colors: Record<Project['status'], string> = {
    planning: 'blue',
    active: 'green',
    on_hold: 'orange',
    completed: 'grey',
  }
  return colors[status]
}

const translateStatus = (status: Project['status']): string => {
  const translations: Record<Project['status'], string> = {
    planning: 'Planejamento',
    active: 'Ativo',
    on_hold: 'Em espera',
    completed: 'Concluído',
  }
  return translations[status]
}
</script>
