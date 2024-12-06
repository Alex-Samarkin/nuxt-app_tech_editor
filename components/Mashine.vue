<template>
  <q-card class="my-card" v-if="mashine">
    <q-card-section>
      <div class="text-h6">{{ mashine.mark }} {{ mashine.model }}</div>
      <div class="text-subtitle2">{{ mashine.company }}</div>
    </q-card-section>

    <q-separator />

    <q-card-section>
      <div class="row q-gutter-md">
        <div class="col-12">
          <div class="text-subtitle2">Basic Information</div>
          <div class="q-gutter-sm">
            <q-badge color="primary">
              Type: {{ mashine.type }}
            </q-badge>
            <q-badge color="secondary">
              Power: {{ mashine.power }}
            </q-badge>
          </div>
        </div>

        <div class="col-12">
          <div class="text-subtitle2">Dimensions</div>
          <div class="q-gutter-sm">
            <q-badge color="grey">
              Length: {{ mashine.dimensions.length }} {{ mashine.dimensions.unit }}
            </q-badge>
            <q-badge color="grey">
              Width: {{ mashine.dimensions.width }} {{ mashine.dimensions.unit }}
            </q-badge>
            <q-badge color="grey">
              Height: {{ mashine.dimensions.height }} {{ mashine.dimensions.unit }}
            </q-badge>
          </div>
        </div>

        <div class="col-12">
          <div class="text-subtitle2">Special Attributes</div>
          <div class="q-gutter-sm">
            <template v-for="(value, key) in mashine.specialAttributes" :key="key">
              <q-badge color="purple">
                {{ key }}: {{ value }}
              </q-badge>
            </template>
          </div>
        </div>
      </div>
    </q-card-section>
  </q-card>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import mashinesData from '@/assets/machines.json'

const props = defineProps({
  mashineId: {
    type: String,
    required: true
  }
})

const mashine = ref(null)

onMounted(() => {
  mashine.value = mashinesData.machines.find(m => m.id === props.mashineId)
})
</script>

<style scoped>
.my-card {
  max-width: 700px;
  margin: 20px auto;
}
</style>
