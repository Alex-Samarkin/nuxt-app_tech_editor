<template>
  <div class="q-pa-md">
    <div class="row items-center justify-between q-mb-md">
      <div class="text-h5">Machine Tools Management</div>
      <q-btn
        color="primary"
        icon="add"
        label="Add New Tool"
        @click="showEditor = true"
      />
    </div>

    <MachineToolsTable v-if="!showEditor" />
    
    <MachineToolEditor
      v-else
      @save="onSave"
      @cancel="showEditor = false"
    />
  </div>
</template>

<script setup>
import { ref } from 'vue'
import MachineToolsTable from '@/components/MachineToolsTable.vue'
import MachineToolEditor from '@/components/MachineToolEditor.vue'

const showEditor = ref(false)

const onSave = (toolData) => {
  // Here you would typically make an API call to save the data
  showEditor.value = false
}
</script>
