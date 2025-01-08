<template>
  <q-item clickable v-ripple @click="$emit('view-task', task)">
    <q-item-section>
      <q-item-label>{{ task.title }}</q-item-label>
      <q-item-label caption>{{ task.description }}</q-item-label>
    </q-item-section>
    <q-item-section side>
      <q-chip :color="getStatusColor(task.status)" text-color="white">
        {{ getStatusLabel(task.status) }}
      </q-chip>
    </q-item-section>
  </q-item>
</template>

<script setup lang="ts">
import type { PropType } from 'vue'

const { task } = defineProps({
  task: {
    type: Object as PropType<{
      id: number
      title: string
      description: string
      status: 'pending' | 'in_progress' | 'completed'
      created_at: string
    }>,
    required: true,
  },
})

const getStatusColor = (status: string) => {
  switch (status) {
    case 'pending':
      return 'orange'
    case 'in_progress':
      return 'blue'
    case 'completed':
      return 'green'
    default:
      return 'grey'
  }
}

const getStatusLabel = (status: string) => {
  switch (status) {
    case 'pending':
      return 'Pendente'
    case 'in_progress':
      return 'Em Andamento'
    case 'completed':
      return 'Concluído'
    default:
      return status
  }
}
</script>
