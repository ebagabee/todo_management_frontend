<template>
  <q-page padding>
    <h1 class="text-h4 q-mb-md">Visão Geral</h1>

    <div class="row q-col-gutter-md">
      <div class="col-12 col-md-4">
        <q-card>
          <q-card-section>
            <div class="text-h6">Projetos em Andamento</div>
            <div class="text-h3 text-primary text-center q-mt-md">
              {{ overview.projects_in_progress }}
            </div>
            <q-tooltip>Total de projetos ativos no momento</q-tooltip>
          </q-card-section>
        </q-card>
      </div>

      <div class="col-12 col-md-4">
        <q-card>
          <q-card-section>
            <div class="text-h6">Total de Tarefas</div>
            <div class="text-h3 text-info text-center q-mt-md">
              {{ overview.total_tasks }}
            </div>
            <q-tooltip>Número total de tarefas em todos os projetos</q-tooltip>
          </q-card-section>
        </q-card>
      </div>

      <div class="col-12 col-md-4">
        <q-card>
          <q-card-section>
            <div class="text-h6">Tarefas Concluídas</div>
            <div class="text-h3 text-positive text-center q-mt-md">
              {{ overview.completed_tasks }}
            </div>
            <q-tooltip>Número de tarefas finalizadas</q-tooltip>
          </q-card-section>
        </q-card>
      </div>
    </div>

    <div class="row q-mt-md">
      <div class="col-12">
        <q-card>
          <q-card-section>
            <div class="text-h6">Progresso de Tarefas</div>
            <q-linear-progress :value="taskCompletionPercentage" color="positive" class="q-mt-md" />
            <div class="text-caption text-center q-mt-sm">
              {{ formattedTaskCompletionPercentage }}% de tarefas concluídas
              <q-icon :name="progressIcon" :color="progressIconColor" class="q-ml-sm" />
            </div>
          </q-card-section>
        </q-card>
      </div>
    </div>
  </q-page>
</template>

<script setup lang="ts">
import { ref, onMounted, computed } from 'vue'
import { api } from 'src/boot/axios'

interface Overview {
  projects_in_progress: number
  total_tasks: number
  completed_tasks: number
}

const overview = ref<Overview>({
  projects_in_progress: 0,
  total_tasks: 0,
  completed_tasks: 0,
})

const taskCompletionPercentage = computed(() => {
  if (overview.value.total_tasks === 0) return 0
  return overview.value.completed_tasks / overview.value.total_tasks
})

const formattedTaskCompletionPercentage = computed(() => {
  return (taskCompletionPercentage.value * 100).toFixed(2)
})

const progressIcon = computed(() => {
  if (taskCompletionPercentage.value < 0.25) return 'sentiment_very_dissatisfied'
  if (taskCompletionPercentage.value < 0.5) return 'sentiment_dissatisfied'
  if (taskCompletionPercentage.value < 0.75) return 'sentiment_satisfied'
  return 'sentiment_very_satisfied'
})

const progressIconColor = computed(() => {
  if (taskCompletionPercentage.value < 0.25) return 'negative'
  if (taskCompletionPercentage.value < 0.5) return 'warning'
  if (taskCompletionPercentage.value < 0.75) return 'info'
  return 'positive'
})

onMounted(async () => {
  try {
    const response = await api.get<Overview>('/overview')
    overview.value = response.data
  } catch (error) {
    console.error('Error fetching overview:', error)
  }
})
</script>

<style scoped>
.q-card {
  height: 200px;
  display: flex;
  flex-direction: column;
  justify-content: center;
}
</style>
