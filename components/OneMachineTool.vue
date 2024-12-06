<template>
  <q-card class="my-card" v-if="machineTool">
    <q-card-section>
      <div class="text-h6">{{ machineTool.manufacturer }} {{ machineTool.model }}</div>
      <div class="text-subtitle2">Type: {{ machineTool.type }}</div>
    </q-card-section>

    <q-separator />

    <q-card-section>
      <div class="row q-gutter-md">
        <!-- Basic Information -->
        <div class="col-12">
          <div class="text-subtitle2">Basic Information</div>
          <div class="q-gutter-sm">
            <q-badge color="primary">
              Material: {{ machineTool.material }}
            </q-badge>
            <q-badge v-if="machineTool.coating" color="secondary">
              Coating: {{ machineTool.coating }}
            </q-badge>
            <q-badge v-if="machineTool.diameter" color="accent">
              Diameter: {{ machineTool.diameter }} mm
            </q-badge>
            <q-badge v-if="machineTool.length" color="positive">
              Length: {{ machineTool.length }} mm
            </q-badge>
            <q-badge v-if="machineTool.flutes" color="negative">
              Flutes: {{ machineTool.flutes }}
            </q-badge>
            <q-badge v-if="machineTool.insert_size" color="info">
              Insert Size: {{ machineTool.insert_size }}
            </q-badge>
          </div>
        </div>

        <!-- Specifications -->
        <div class="col-12">
          <div class="text-subtitle2">Specifications</div>
          <div class="q-gutter-sm">
            <template v-for="(value, key) in machineTool.specifications" :key="key">
              <q-badge color="purple">
                {{ formatSpecKey(key) }}: {{ value }}
              </q-badge>
            </template>
          </div>
        </div>

        <!-- Compatible Machines -->
        <div class="col-12">
          <div class="text-subtitle2">Compatible Machines</div>
          <div class="q-gutter-sm">
            <q-badge 
              v-for="machineId in machineTool.compatibility" 
              :key="machineId"
              color="teal"
              class="cursor-pointer"
              @click="viewMachine(machineId)"
            >
              {{ getMachineName(machineId) }}
            </q-badge>
          </div>
        </div>
      </div>
    </q-card-section>

    <!-- Machine Details Dialog -->
    <q-dialog v-model="showMachineDialog">
      <q-card style="min-width: 350px">
        <Mashine :mashineId="selectedMachineId" />
      </q-card>
    </q-dialog>
  </q-card>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import machineToolsData from '@/assets/machineTools.json'
import mashinesData from '@/assets/machines.json'
import Mashine from './Mashine.vue'

const props = defineProps({
  machineToolId: {
    type: String,
    required: true
  }
})

const machineTool = ref(null)
const showMachineDialog = ref(false)
const selectedMachineId = ref('')

onMounted(() => {
  machineTool.value = machineToolsData.machineTools.find(mt => mt.id === props.machineToolId)
})

const formatSpecKey = (key) => {
  return key.split('_').map(word => word.charAt(0).toUpperCase() + word.slice(1)).join(' ')
}

const getMachineName = (machineId) => {
  const machine = mashinesData.machines.find(m => m.id === machineId)
  return machine ? `${machine.mark} ${machine.model}` : machineId
}

const viewMachine = (machineId) => {
  selectedMachineId.value = machineId
  showMachineDialog.value = true
}
</script>

<style scoped>
.my-card {
  max-width: 700px;
  margin: 20px auto;
}

.cursor-pointer {
  cursor: pointer;
}
</style>
