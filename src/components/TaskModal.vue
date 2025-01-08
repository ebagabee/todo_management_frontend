<template>
  <q-dialog
    :model-value="modelValue"
    @update:model-value="$emit('update:modelValue', $event)"
    persistent
  >
    <q-card style="min-width: 350px">
      <q-card-section>
        <div class="text-h6">{{ isEditing ? 'Editar Tarefa' : 'Detalhes da Tarefa' }}</div>
      </q-card-section>

      <q-card-section class="q-pt-none">
        <q-form @submit.prevent="saveTask">
          <q-input
            v-model="editedTask.title"
            label="Título"
            :readonly="!isEditing"
            :rules="[(val) => !!val || 'Título é obrigatório']"
          />
          <q-input
            v-model="editedTask.description"
            label="Descrição"
            type="textarea"
            :readonly="!isEditing"
          />
          <q-select
            v-model="editedTask.status"
            :options="statusOptions"
            label="Status"
            :readonly="!isEditing"
            emit-value
            map-options
          />
        </q-form>
      </q-card-section>

      <q-card-actions align="right">
        <q-btn flat label="Fechar" color="primary" @click="$emit('update:modelValue', false)" />
        <q-btn v-if="!isEditing" flat label="Editar" color="primary" @click="isEditing = true" />
        <q-btn v-else flat label="Salvar" color="primary" @click="saveTask" />
      </q-card-actions>
    </q-card>
  </q-dialog>
</template>

<script setup lang="ts">
import type { PropType } from 'vue'
import { ref, watch } from 'vue'

const props = defineProps({
  modelValue: {
    type: Boolean,
    required: true,
  },
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

const emit = defineEmits<{
  (e: 'update:modelValue', value: boolean): void
  (
    e: 'update-task',
    task: {
      id: number
      title: string
      description: string
      status: 'pending' | 'in_progress' | 'completed'
    },
  ): void
}>()

const editedTask = ref({ ...props.task })
const isEditing = ref(false)
const statusOptions = [
  { value: 'pending', label: 'Pendente' },
  { value: 'in_progress', label: 'Em Andamento' },
  { value: 'completed', label: 'Concluído' },
]

watch(
  () => props.task,
  (newTask) => {
    editedTask.value = { ...newTask }
    isEditing.value = false
  },
  { deep: true },
)

const saveTask = () => {
  if (editedTask.value.title.trim()) {
    emit('update-task', {
      id: editedTask.value.id,
      title: editedTask.value.title,
      description: editedTask.value.description,
      status: editedTask.value.status,
    })
    isEditing.value = false
    emit('update:modelValue', false)
  }
}
</script>
