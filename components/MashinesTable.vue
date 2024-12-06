<template>
  <div class="q-pa-md">
    <q-table
      title="Machines"
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
      <!-- Top section with search, export, and column selection -->
      <template v-slot:top>
        <div class="row full-width">
          <div class="col-6 q-table__title">Machines</div>
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
            v-model="companyFilter"
            :options="companyOptions"
            label="Company"
            outlined
            dense
            clearable
            options-dense
            style="min-width: 150px"
            @update:model-value="applyFilters"
          />
          <q-input
            v-model="powerFilter"
            type="number"
            label="Min Power (kW)"
            outlined
            dense
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

      <!-- Custom pagination -->
      <template v-slot:pagination="props">
        <q-pagination
          v-model="props.pagination.page"
          :max="Math.ceil(filteredRows.length / props.pagination.rowsPerPage)"
          :max-pages="6"
          boundary-links
          direction-links
          @update:model-value="updatePage"
        />
        <q-select
          v-model="props.pagination.rowsPerPage"
          :options="[5, 10, 20, 50]"
          label="Rows per page"
          outlined
          dense
          options-dense
          style="min-width: 120px; margin-left: 10px"
          @update:model-value="updateRowsPerPage"
        />
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
              @click="viewMachine(props.row)"
            >
              <q-tooltip>View Details</q-tooltip>
            </q-btn>
            <q-btn
              color="secondary"
              icon="edit"
              dense
              flat
              @click="editMachine(props.row)"
            >
              <q-tooltip>Edit</q-tooltip>
            </q-btn>
            <q-btn
              color="negative"
              icon="delete"
              dense
              flat
              @click="deleteMachine(props.row)"
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
        <Mashine :mashineId="selectedMachineId" />
      </q-card>
    </q-dialog>

    <!-- Compare Dialog -->
    <q-dialog v-model="showCompareDialog" full-width>
      <q-card>
        <q-card-section>
          <div class="text-h6">Compare Machines</div>
        </q-card-section>
        <q-card-section class="row q-gutter-md">
          <Mashine v-for="machine in selected" :key="machine.id" :mashineId="machine.id" />
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
import mashinesData from '@/assets/machines.json'
import Mashine from './Mashine.vue'
import { useQuasar } from 'quasar'
import { saveAs } from 'file-saver'
import * as XLSX from 'xlsx'

const $q = useQuasar()

const machines = ref([])
const loading = ref(true)
const filter = ref('')
const showDialog = ref(false)
const showCompareDialog = ref(false)
const selectedMachineId = ref('')
const selected = ref([])
const visibleColumns = ref(['id', 'type', 'mark', 'model', 'company', 'power', 'dimensions', 'actions'])
const typeFilter = ref(null)
const companyFilter = ref(null)
const powerFilter = ref(null)

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
    name: 'mark',
    required: true,
    label: 'Mark',
    align: 'left',
    field: row => row.mark,
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
    name: 'company',
    required: true,
    label: 'Company',
    align: 'left',
    field: row => row.company,
    sortable: true
  },
  {
    name: 'power',
    required: true,
    label: 'Power',
    align: 'left',
    field: row => row.power,
    sortable: true
  },
  {
    name: 'dimensions',
    required: true,
    label: 'Dimensions (L×W×H)',
    align: 'left',
    field: row => `${row.dimensions.length}×${row.dimensions.width}×${row.dimensions.height} ${row.dimensions.unit}`,
    sortable: false
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
  return [...new Set(machines.value.map(m => m.type))]
})

const companyOptions = computed(() => {
  return [...new Set(machines.value.map(m => m.company))]
})

const filteredRows = computed(() => {
  let filtered = [...machines.value]

  // Apply type filter
  if (typeFilter.value) {
    filtered = filtered.filter(machine => machine.type === typeFilter.value)
  }

  // Apply company filter
  if (companyFilter.value) {
    filtered = filtered.filter(machine => machine.company === companyFilter.value)
  }

  // Apply power filter
  if (powerFilter.value) {
    filtered = filtered.filter(machine => {
      const machinePower = parseInt(machine.power)
      return !isNaN(machinePower) && machinePower >= powerFilter.value
    })
  }

  // Apply global search filter
  if (filter.value) {
    const searchText = filter.value.toLowerCase()
    filtered = filtered.filter(machine => {
      return Object.values(machine).some(value => {
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
    : `${selected.value.length} record${selected.value.length > 1 ? 's' : ''} selected of ${machines.value.length}`
}

const viewMachine = (machine) => {
  selectedMachineId.value = machine.id
  showDialog.value = true
}

const editMachine = (machine) => {
  // Implement edit functionality
  $q.notify({
    message: `Edit machine ${machine.id}`,
    color: 'info'
  })
}

const deleteMachine = (machine) => {
  $q.dialog({
    title: 'Confirm',
    message: `Are you sure you want to delete machine ${machine.id}?`,
    cancel: true,
    persistent: true
  }).onOk(() => {
    // Implement delete functionality
    $q.notify({
      message: `Deleted machine ${machine.id}`,
      color: 'negative'
    })
  })
}

const viewSelected = () => {
  if (selected.value.length === 1) {
    viewMachine(selected.value[0])
  } else {
    showCompareDialog.value = true
  }
}

const deleteSelected = () => {
  $q.dialog({
    title: 'Confirm',
    message: `Are you sure you want to delete ${selected.value.length} machines?`,
    cancel: true,
    persistent: true
  }).onOk(() => {
    // Implement bulk delete functionality
    $q.notify({
      message: `Deleted ${selected.value.length} machines`,
      color: 'negative'
    })
  })
}

const compareSelected = () => {
  showCompareDialog.value = true
}

const exportTable = (format) => {
  const exportData = selected.value.length ? selected.value : machines.value
  const filteredData = exportData.map(({ id, type, mark, model, company, power }) => ({
    id, type, mark, model, company, power
  }))

  if (format === 'csv') {
    const csv = convertToCSV(filteredData)
    const blob = new Blob([csv], { type: 'text/csv;charset=utf-8' })
    saveAs(blob, 'machines.csv')
  } else if (format === 'excel') {
    const worksheet = XLSX.utils.json_to_sheet(filteredData)
    const workbook = XLSX.utils.book_new()
    XLSX.utils.book_append_sheet(workbook, worksheet, 'Machines')
    const excelBuffer = XLSX.write(workbook, { bookType: 'xlsx', type: 'array' })
    const blob = new Blob([excelBuffer], { type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet' })
    saveAs(blob, 'machines.xlsx')
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

const updatePage = (page) => {
  pagination.value.page = page
}

const updateRowsPerPage = (rowsPerPage) => {
  pagination.value.rowsPerPage = rowsPerPage
}

const applyFilters = () => {
  pagination.value.page = 1
}

onMounted(() => {
  machines.value = mashinesData.machines
  loading.value = false
})
</script>

<style scoped>
.q-table {
  background: white;
}
</style>
