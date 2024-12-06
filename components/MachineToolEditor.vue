<template>
  <q-card class="editor-card">
    <q-card-section>
      <div class="text-h6">{{ isEditing ? 'Edit Machine Tool' : 'Add New Machine Tool' }}</div>
    </q-card-section>

    <q-card-section>
      <q-form @submit="onSubmit" class="q-gutter-md">
        <!-- Basic Information -->
        <div class="row q-col-gutter-md">
          <div class="col-12 col-md-6">
            <q-input
              v-model="form.type"
              label="Type *"
              :rules="[val => !!val || 'Type is required']"
              outlined
            />
          </div>
          <div class="col-12 col-md-6">
            <q-input
              v-model="form.manufacturer"
              label="Manufacturer *"
              :rules="[val => !!val || 'Manufacturer is required']"
              outlined
            />
          </div>
          <div class="col-12 col-md-6">
            <q-input
              v-model="form.model"
              label="Model *"
              :rules="[val => !!val || 'Model is required']"
              outlined
            />
          </div>
          <div class="col-12 col-md-6">
            <q-input
              v-model="form.material"
              label="Material *"
              :rules="[val => !!val || 'Material is required']"
              outlined
            />
          </div>
          <div class="col-12 col-md-6">
            <q-input
              v-model="form.coating"
              label="Coating"
              outlined
            />
          </div>
        </div>

        <!-- Dimensions -->
        <div class="row q-col-gutter-md">
          <div class="col-12 col-md-6">
            <q-input
              v-model.number="form.diameter"
              type="number"
              label="Diameter (mm)"
              outlined
            />
          </div>
          <div class="col-12 col-md-6">
            <q-input
              v-model.number="form.length"
              type="number"
              label="Length (mm)"
              outlined
            />
          </div>
          <div class="col-12 col-md-6">
            <q-input
              v-model.number="form.flutes"
              type="number"
              label="Number of Flutes"
              outlined
            />
          </div>
        </div>

        <!-- Specifications -->
        <q-expansion-item
          group="specs"
          icon="settings"
          label="Specifications"
          header-class="text-primary"
        >
          <q-card>
            <q-card-section>
              <div class="row q-col-gutter-md">
                <div class="col-12 col-md-6">
                  <q-input
                    v-model="form.specifications.cutting_speed"
                    label="Cutting Speed"
                    outlined
                  />
                </div>
                <div class="col-12 col-md-6">
                  <q-input
                    v-model="form.specifications.feed_rate"
                    label="Feed Rate"
                    outlined
                  />
                </div>
                <div class="col-12 col-md-6">
                  <q-input
                    v-model="form.specifications.max_depth_of_cut"
                    label="Max Depth of Cut"
                    outlined
                  />
                </div>
              </div>
            </q-card-section>
          </q-card>
        </q-expansion-item>

        <!-- Machine Compatibility -->
        <q-expansion-item
          group="specs"
          icon="build"
          label="Compatible Machines"
          header-class="text-primary"
        >
          <q-card>
            <q-card-section>
              <q-select
                v-model="form.compatibility"
                :options="machineOptions"
                label="Compatible Machines"
                multiple
                use-chips
                outlined
              />
            </q-card-section>
          </q-card>
        </q-expansion-item>

        <!-- Submit Buttons -->
        <div class="row justify-end q-gutter-sm">
          <q-btn
            label="Cancel"
            color="negative"
            flat
            @click="$emit('cancel')"
          />
          <q-btn
            type="submit"
            :label="isEditing ? 'Save Changes' : 'Add Tool'"
            color="primary"
          />
        </div>
      </q-form>
    </q-card-section>
  </q-card>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'
import { useQuasar } from 'quasar'
import machineToolsData from '@/assets/machineTools.json'
import mashinesData from '@/assets/machines.json'

const props = defineProps({
  machineToolId: {
    type: String,
    default: null
  }
})

const emit = defineEmits(['save', 'cancel'])
const $q = useQuasar()

const isEditing = computed(() => !!props.machineToolId)

const defaultForm = {
  type: '',
  manufacturer: '',
  model: '',
  material: '',
  coating: '',
  diameter: null,
  length: null,
  flutes: null,
  specifications: {
    cutting_speed: '',
    feed_rate: '',
    max_depth_of_cut: ''
  },
  compatibility: []
}

const form = ref({ ...defaultForm })

const machineOptions = computed(() => {
  return mashinesData.machines.map(machine => ({
    label: `${machine.mark} ${machine.model}`,
    value: machine.id
  }))
})

onMounted(() => {
  if (props.machineToolId) {
    const machineTool = machineToolsData.machineTools.find(mt => mt.id === props.machineToolId)
    if (machineTool) {
      form.value = { ...machineTool }
    }
  }
})

const onSubmit = () => {
  // Here you would typically make an API call to save the data
  $q.notify({
    message: isEditing.value ? 'Machine tool updated successfully' : 'Machine tool added successfully',
    color: 'positive'
  })
  emit('save', form.value)
}
</script>

<style scoped>
.editor-card {
  max-width: 900px;
  margin: 20px auto;
}
</style>
