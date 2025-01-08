<template>
  <q-form @submit.prevent="submitTask" class="q-gutter-md">
    <q-input v-model="task.title" filled label="Título" />
    <q-input v-model="task.description" filled type="textarea" label="Descrição" />
    <q-select
      v-model="task.status"
      :options="statusOptions"
      filled
      label="Status"
      emit-value
      map-options
    />
    <q-btn label="Adicionar Tarefa" type="submit" color="primary" />
  </q-form>
</template>

<script setup lang="ts">
import { ref } from 'vue'

const emit = defineEmits<{
  (
    e: 'add-task',
    task: { title: string; description: string; status: 'pending' | 'in_progress' | 'completed' },
  ): void
}>()

const statusOptions = [
  { value: 'pending', label: 'Pendente' },
  { value: 'in_progress', label: 'Em Andamento' },
  { value: 'completed', label: 'Concluído' },
]

const task = ref({
  title: '',
  description: '',
  status: 'pending' as 'pending' | 'in_progress' | 'completed',
})

const submitTask = () => {
  if (task.value.title.trim()) {
    emit('add-task', { ...task.value })
    task.value = { title: '', description: '', status: 'pending' }
  }
}
</script>
