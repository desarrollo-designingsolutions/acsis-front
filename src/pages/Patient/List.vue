<script setup lang="ts">
import { router } from '@/plugins/1.router';
import { useAuthenticationStore } from "@/stores/useAuthenticationStore";

definePage({
  name: "Patient-List",
  meta: {
    redirectIfLoggedIn: true,
    requiresAuth: true,
    requiredPermission: "menu.patient",
  },
});

const loading = reactive({ excel: false })
const route = useRoute()

const authenticationStore = useAuthenticationStore();

//FILTER
const optionsFilter = ref({
  dialog: {
    cols: "12",
    width: 500,
    inputs: [
      {
        name: "is_active",
        label: "Estado",
        type: "booleanActive",
        value: null,
        placeholder: "Ingrese valor"
      },
    ],
  },
})

//TABLE
const refTableFull = ref()
const optionsTable = {
  url: "/patient/paginate",
  paramsGlobal: {
    company_id: authenticationStore.company.id,
  },
  headers: [
    { key: 'document', title: 'Documento' },
    { key: 'full_name', title: 'Nombre' },
    { key: 'actions', title: 'Acciones', sortable: false, width: 50 },
  ],
  actions: {
    changeStatus: {
      url: "/patient/changeStatus"
    },
    view: {
      show: true,
    },
    delete: {
      url: "/patient/delete",
      show: true
    },
  }
}

const goViewEdit = (data: any) => {
  router.push({ name: "Patient-Form", params: { action: "edit", id: data.id } })
}

const goViewCreate = () => {
  router.push({ name: "Patient-Form", params: { action: "create" } })
}

const goViewView = (data: any) => {
  router.push({ name: "Patient-Form", params: { action: "view", id: data.id } })
}

const tableLoading = ref(false); // Estado de carga de la tabla

// Método para refrescar los datos
const refreshTable = () => {
  if (refTableFull.value) {
    refTableFull.value.fetchTableData(null, false, true); // Forzamos la búsqueda
  }
};

const downloadExcel = async () => {
  loading.excel = true;

  const { data, response } = await useAxios("/patient/excelExport").get({
    params: {
      ...route.query,
      company_id: authenticationStore.company.id
    }
  })

  loading.excel = false;

  if (response.status == 200 && data) {
    downloadExcelBase64(data.excel, "Lista de Pacientes")
  }
}

</script>

<template>
  <div>

    <VCard>
      <VCardTitle class="d-flex justify-space-between">
        <span>
          Pacientes
        </span>

        <div class="d-flex justify-end gap-3 flex-wrap ">
          <VBtn :loading="loading.excel" :disabled="loading.excel" size="38" color="primary" icon
            @click="downloadExcel()">
            <VIcon icon="tabler-file-spreadsheet"></VIcon>
            <VTooltip location="top" transition="scale-transition" activator="parent" text="Descargar Excel">
            </VTooltip>
          </VBtn>

          <VBtn @click="goViewCreate()">
            Agregar Paciente
          </VBtn>
        </div>
      </VCardTitle>

      <!-- Sección de filtros mejorada -->
      <VCardText>
        <FilterDialog :options-filter="optionsFilter" @force-search="refreshTable" :table-loading="tableLoading">
        </FilterDialog>
      </VCardText>

      <VCardText class="mt-2">

        <TableFull ref="refTableFull" :options="optionsTable" @edit="goViewEdit" @view="goViewView"
          @update:loading="tableLoading = $event">



        </TableFull>


      </VCardText>
    </VCard>
  </div>
</template>
