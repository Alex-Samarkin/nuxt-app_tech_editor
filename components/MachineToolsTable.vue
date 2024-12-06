<template>
  <div class="q-pa-md">
    <q-table
      title="Machine Tools"
      :rows="filteredRows"
      :columns="columns"
      row-key="id"
      :filter="filter"
      :loading="loading"
      :pagination.sync="pagination"
      :selected-rows-label="getSelectedString"
      selection="multiple"
      v-model:selected="selected"
      :visible-columns="visibleColumns"
    >
      <!-- Top section -->
      <template v-slot:top>
        <div class="row full-width">
          <div class="col-6 q-table__title">Machine Tools</div>
          <div class="col-6 row items-center justify-end q-gutter-sm">
            <!-- Search input -->
            <q-input
              borderless
              dense
              debounce="300"
              v-model="filter"
              placeholder="Search"
            >
              <template v-slot:append>
                <q-icon name="search" />
              </template>
            </q-input>

            <!-- Column selection -->
            <q-select
              v-model="visibleColumns"
              multiple
              outlined
              dense
              options-dense
              :display-value="$q.lang.table.columns"
              emit-value
              map-options
              :options="columns"
              option-value="name"
              option-label="label"
              options-cover
              style="min-width: 150px"
            />

            <!-- Export button -->
            <q-btn-dropdown color="primary" label="Export" icon="get_app">
              <q-list>
                <q-item clickable v-close-popup @click="exportTable('csv')">
                  <q-item-section>
                    <q-item-label>Export as CSV</q-item-label>
                  </q-item-section>
                </q-item>

                <q-item clickable v-close-popup @click="exportTable('excel')">
                  <q-item-section>
                    <q-item-label>Export as Excel</q-item-label>
                  </q-item-section>
                </q-item>
              </q-list>
            </q-btn-dropdown>
          </div>
        </div>

        <!-- Advanced filters -->
        <div class="row q-mt-sm q-gutter-sm">
          <q-select
            v-model="typeFilter"
            :options="typeOptions"
            label="Type"
            outlined
            dense
            clearable
            options-dense
            style="min-width: 150px"
            @update:model-value="applyFilters"
          />
          <q-select
            v-model="manufacturerFilter"
            :options="manufacturerOptions"
            label="Manufacturer"
            outlined
            dense
            clearable
            options-dense
            style="min-width: 150px"
            @update:model-value="applyFilters"
          />
          <q-select
            v-model="materialFilter"
            :options="materialOptions"
            label="Material"
            outlined
            dense
            clearable
            options-dense
            style="min-width: 150px"
            @update:model-value="applyFilters"
          />
        </div>

        <!-- Bulk actions -->
        <div v-if="selected.length" class="row q-mt-sm">
          <q-btn-group spread>
            <q-btn color="primary" icon="visibility" label="View Selected" @click="viewSelected" />
            <q-btn color="negative" icon="delete" label="Delete Selected" @click="deleteSelected" />
            <q-btn color="secondary" icon="compare_arrows" label="Compare Selected" @click="compareSelected" />
          </q-btn-group>
        </div>
      </template>

      <!-- Actions column -->
      <template v-slot:body-cell-actions="props">
        <q-td :props="props">
          <q-btn-group spread flat>
            <q-btn
              color="primary"
              icon="visibility"
              dense
              flat
              @click="viewMachineTool(props.row)"
            >
              <q-tooltip>View Details</q-tooltip>
            </q-btn>
            <q-btn
              color="secondary"
              icon="edit"
              dense
              flat
              @click="editMachineTool(props.row)"
            >
              <q-tooltip>Edit</q-tooltip>
            </q-btn>
            <q-btn
              color="negative"
              icon="delete"
              dense
              flat
              @click="deleteMachineTool(props.row)"
            >
              <q-tooltip>Delete</q-tooltip>
            </q-btn>
          </q-btn-group>
        </q-td>
      </template>
    </q-table>

    <!-- View Dialog -->
    <q-dialog v-model="showDialog">
      <q-card style="min-width: 350px">
        <OneMachineTool :machineToolId="selectedMachineToolId" />
      </q-card>
    </q-dialog>

    <!-- Compare Dialog -->
    <q-dialog v-model="showCompareDialog" full-width>
      <q-card>
        <q-card-section>
          <div class="text-h6">Compare Machine Tools</div>
        </q-card-section>
        <q-card-section class="row q-gutter-md">
          <OneMachineTool 
            v-for="tool in selected" 
            :key="tool.id" 
            :machineToolId="tool.id" 
          />
        </q-card-section>
        <q-card-actions align="right">
          <q-btn flat label="Close" color="primary" v-close-popup />
        </q-card-actions>
      </q-card>
    </q-dialog>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import machineToolsData from '@/assets/machineTools.json'
import OneMachineTool from './OneMachineTool.vue'
import { useQuasar } from 'quasar'
import { saveAs } from 'file-saver'
import * as XLSX from 'xlsx'

const $q = useQuasar()

const machineTools = ref([])
const loading = ref(true)
const filter = ref('')
const showDialog = ref(false)
const showCompareDialog = ref(false)
const selectedMachineToolId = ref('')
const selected = ref([])
const visibleColumns = ref(['id', 'type', 'manufacturer', 'model', 'material', 'coating', 'actions'])
const typeFilter = ref(null)
const manufacturerFilter = ref(null)
const materialFilter = ref(null)

const pagination = ref({
  sortBy: 'id',
  descending: false,
  page: 1,
  rowsPerPage: 10
})

// Column definitions
const columns = [
  {
    name: 'id',
    required: true,
    label: 'ID',
    align: 'left',
    field: row => row.id,
    sortable: true
  },
  {
    name: 'type',
    required: true,
    label: 'Type',
    align: 'left',
    field: row => row.type,
    sortable: true
  },
  {
    name: 'manufacturer',
    required: true,
    label: 'Manufacturer',
    align: 'left',
    field: row => row.manufacturer,
    sortable: true
  },
  {
    name: 'model',
    required: true,
    label: 'Model',
    align: 'left',
    field: row => row.model,
    sortable: true
  },
  {
    name: 'material',
    required: true,
    label: 'Material',
    align: 'left',
    field: row => row.material,
    sortable: true
  },
  {
    name: 'coating',
    required: false,
    label: 'Coating',
    align: 'left',
    field: row => row.coating,
    sortable: true
  },
  {
    name: 'diameter',
    required: false,
    label: 'Diameter (mm)',
    align: 'left',
    field: row => row.diameter,
    sortable: true
  },
  {
    name: 'actions',
    required: true,
    label: 'Actions',
    align: 'center',
    field: row => row.id,
    sortable: false
  }
]

// Computed properties for filters
const typeOptions = computed(() => {
  return [...new Set(machineTools.value.map(mt => mt.type))]
})

const manufacturerOptions = computed(() => {
  return [...new Set(machineTools.value.map(mt => mt.manufacturer))]
})

const materialOptions = computed(() => {
  return [...new Set(machineTools.value.map(mt => mt.material))]
})

const filteredRows = computed(() => {
  let filtered = [...machineTools.value]

  if (typeFilter.value) {
    filtered = filtered.filter(tool => tool.type === typeFilter.value)
  }

  if (manufacturerFilter.value) {
    filtered = filtered.filter(tool => tool.manufacturer === manufacturerFilter.value)
  }

  if (materialFilter.value) {
    filtered = filtered.filter(tool => tool.material === materialFilter.value)
  }

  if (filter.value) {
    const searchText = filter.value.toLowerCase()
    filtered = filtered.filter(tool => {
      return Object.values(tool).some(value => {
        if (typeof value === 'string') {
          return value.toLowerCase().includes(searchText)
        }
        if (typeof value === 'object' && value !== null) {
          return Object.values(value).some(v => 
            String(v).toLowerCase().includes(searchText)
          )
        }
        return String(value).toLowerCase().includes(searchText)
      })
    })
  }

  return filtered
})

// Methods
const getSelectedString = () => {
  return selected.value.length === 0
    ? ''
    : `${selected.value.length} record${selected.value.length > 1 ? 's' : ''} selected of ${machineTools.value.length}`
}

const viewMachineTool = (tool) => {
  selectedMachineToolId.value = tool.id
  showDialog.value = true
}

const editMachineTool = (tool) => {
  $q.notify({
    message: `Edit machine tool ${tool.id}`,
    color: 'info'
  })
}

const deleteMachineTool = (tool) => {
  $q.dialog({
    title: 'Confirm',
    message: `Are you sure you want to delete machine tool ${tool.id}?`,
    cancel: true,
    persistent: true
  }).onOk(() => {
    $q.notify({
      message: `Deleted machine tool ${tool.id}`,
      color: 'negative'
    })
  })
}

const viewSelected = () => {
  if (selected.value.length === 1) {
    viewMachineTool(selected.value[0])
  } else {
    showCompareDialog.value = true
  }
}

const deleteSelected = () => {
  $q.dialog({
    title: 'Confirm',
    message: `Are you sure you want to delete ${selected.value.length} machine tools?`,
    cancel: true,
    persistent: true
  }).onOk(() => {
    $q.notify({
      message: `Deleted ${selected.value.length} machine tools`,
      color: 'negative'
    })
  })
}

const compareSelected = () => {
  showCompareDialog.value = true
}

const exportTable = (format) => {
  const exportData = selected.value.length ? selected.value : machineTools.value
  const filteredData = exportData.map(({ id, type, manufacturer, model, material, coating }) => ({
    id, type, manufacturer, model, material, coating
  }))

  if (format === 'csv') {
    const csv = convertToCSV(filteredData)
    const blob = new Blob([csv], { type: 'text/csv;charset=utf-8' })
    saveAs(blob, 'machine-tools.csv')
  } else if (format === 'excel') {
    const worksheet = XLSX.utils.json_to_sheet(filteredData)
    const workbook = XLSX.utils.book_new()
    XLSX.utils.book_append_sheet(workbook, worksheet, 'Machine Tools')
    const excelBuffer = XLSX.write(workbook, { bookType: 'xlsx', type: 'array' })
    const blob = new Blob([excelBuffer], { type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet' })
    saveAs(blob, 'machine-tools.xlsx')
  }
}

const convertToCSV = (data) => {
  const headers = Object.keys(data[0])
  const csvRows = [
    headers.join(','),
    ...data.map(row => headers.map(header => row[header]).join(','))
  ]
  return csvRows.join('\n')
}

onMounted(() => {
  machineTools.value = machineToolsData.machineTools
  loading.value = false
})
</script>

<style scoped>
.q-table {
  background: white;
}
</style>
