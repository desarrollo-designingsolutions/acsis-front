<script setup lang="ts">
import { router } from '@/plugins/1.router';
import { useAuthenticationStore } from "@/stores/useAuthenticationStore";

definePage({
  name: "Invoice-List",
  meta: {
    redirectIfLoggedIn: true,
    requiresAuth: true,
    requiredPermission: "menu.invoice",
  },
});

const loading = reactive({ excel: false, btnCreate: false })
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
      {
        type: "dateRange",
        label: "Fecha Radicación",
        name: "invoices.radication_date",
      },
    ],
  },
})

//TABLE
const refTableFull = ref()
const optionsTable = {
  url: "/invoice/paginate",
  paramsGlobal: {
    company_id: authenticationStore.company.id,
  },
  headers: [
    { key: 'entity_name', title: 'Entidad' },
    { key: 'invoice_number', title: 'Factura No.' },
    { key: 'type_name', title: 'Tipo Factura' },
    { key: "value_paid", title: 'Valor Pagado', width: '150px', minWidth: '100px' },
    { key: "value_glosa", title: 'Valor Glosa' },
    { key: "radication_date", title: 'Fecha Radicación', },
    { key: "patient_name", title: 'Paciente', },
    { key: "is_active", title: 'Estado', },
    { key: 'actions', title: 'Acciones', sortable: false },
  ],
  actions: {
    changeStatus: {
      url: "/invoice/changeStatus"
    },
    view: {
      show: true,
    },
    delete: {
      url: "/invoice/delete",
      show: true
    },
  }
}

const goViewEdit = (data: any) => {
  router.push({ name: "Invoice-Form", params: { action: "edit", type: data.type_id, id: data.id } })
}

const goViewCreate = (type: any) => {
  router.push({ name: "Invoice-Form", params: { action: "create", type: type.invoice_type_id } })
}

const goViewView = (data: any) => {
  router.push({ name: "Invoice-Form", params: { action: "view", id: data.id } })
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

  const { data, response } = await useAxios("/invoice/excelExport").get({
    params: {
      ...route.query,
      company_id: authenticationStore.company.id
    }
  })

  loading.excel = false;

  if (response.status == 200 && data) {
    downloadExcelBase64(data.excel, "Lista de Facturas")
  }
}

const invoiceTypeBtn = ref([]);

const fetchDataBtn = async () => {
  loading.btnCreate = true

  const { data, response } = await useAxios("/invoice/loadBtnCreate").get()

  if (response.status == 200 && data) {
    invoiceTypeBtn.value = data.TypeInvoiceEnumValues;
  }
  loading.btnCreate = false
}

onMounted(() => {
  fetchDataBtn();
})

</script>

<template>
  <div>

    <VCard>
      <VCardTitle class="d-flex justify-space-between">
        <span>
          Gestión de Facturación y Auditoría
        </span>

        <div class="d-flex justify-end gap-3 flex-wrap ">
          <VBtn :loading="loading.excel" :disabled="loading.excel" size="38" color="primary" icon
            @click="downloadExcel()">
            <VIcon icon="tabler-file-spreadsheet"></VIcon>
            <VTooltip location="top" transition="scale-transition" activator="parent" text="Descargar Excel">
            </VTooltip>
          </VBtn>

          <VBtn color="primary" append-icon="tabler-chevron-down" :loading="loading.btnCreate"
            :disabled="loading.btnCreate">
            Registrar Factura
            <VMenu activator="parent">
              <VList>
                <VListItem v-for="(item, index) in invoiceTypeBtn" :key="index"
                  @click="goViewCreate({ invoice_type_id: item.id })">
                  {{ item.name }}
                </VListItem>
              </VList>
            </VMenu>
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
