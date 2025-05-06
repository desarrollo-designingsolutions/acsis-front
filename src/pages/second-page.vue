<script setup lang="ts">
import ModalFormMasiveGlosa from "@/pages/Glosa/Components/ModalFormMasive.vue";
import ModalListGlosa from "@/pages/Glosa/Components/ModalListGlosa.vue";
import ModalListInvoicePayment from "@/pages/InvoicePayment/Components/ModalListInvoicePayment.vue";
import ModalFormService from "@/pages/ModalFormService.vue";
import { useAuthenticationStore } from "@/stores/useAuthenticationStore";

const { toast } = useToast();

const authenticationStore = useAuthenticationStore();
const invoice_id = "1111"
const servicesIds = ref<Array<string>>([]);

//FILTER
const refFilterDialog = ref()

const optionsFilter = ref({
  filterLabels: { inputGeneral: 'Buscar en todo' }
})

//TABLE
const refTableFull = ref()

const optionsTable = {
  url: `/service/paginate`,
  headers: [
    { key: 'cups_rip_codigo', title: 'Código' },
    { key: 'cups_rip_nombre', title: 'Descripcion' },
    { key: 'quantity', title: 'Cantidad' },
    { key: "unit_value", title: 'Valor Unitario' },
    { key: "total_value", title: 'Valor Total' },
    { key: 'actions', title: 'Acciones', sortable: false },
  ],
  showSelect: true,
  paramsGlobal: {
    company_id: authenticationStore.company.id,
    user_id: authenticationStore.user.id,
    invoice_id: invoice_id,
  },
  actions: {
    delete: {
      url: '/service/delete',
    },
  }
}


//ModalFormService
const refModalFormService = ref()

const openModalFormServiceCreate = () => {
  refModalFormService.value.openModal({ invoice_id: invoice_id })
}

const openModalFormServiceEdit = async (data: any) => {
  refModalFormService.value.openModal({ invoice_id: invoice_id, id: data.id })
}

const openModalFormServiceView = async (data: any) => {
  refModalFormService.value.openModal({ id: data.id }, true)
}

const tableLoading = ref(false); // Estado de carga de la tabla


// Nueva prop para controlar si se actualiza la URL
const disableUrlUpdate = ref(true);

// Nuevo método para manejar la búsqueda forzada desde el filtro
const handleForceSearch = (params) => {
  if (refTableFull.value) {
    // Si disableUrlUpdate está activo, pasamos los parámetros manualmente
    if (disableUrlUpdate.value && params) {
      refTableFull.value.fetchTableData(null, false, true, params);
    } else {
      refTableFull.value.fetchTableData(null, false, true);
    }
  }
};


//ModalListGlosa
const refModalListGlosa = ref()

const openModalListGlosa = (data: any) => {
  refModalListGlosa.value.openModal({
    ...data,
  })
}

//ModalFormMasiveGlosa
const refModalFormMasiveGlosa = ref()

const openModalFormMasiveGlosa = () => {
  if (servicesIds.value.length > 0) {
    refModalFormMasiveGlosa.value.openModal(servicesIds.value)
  } else {
    toast("Debe seleccionar almenos un servicio", "", "info")
  }
}

//ModalListInvoicePayment
const refModalListInvoicePayment = ref()

const openModalListInvoicePayment = () => {
  refModalListInvoicePayment.value.openModal({ invoice_id: invoice_id })
}
</script>

<template>
  <div>

    <VCard>
      <VCardTitle class="d-flex justify-space-between">
        <span>
          Servicios
        </span>

        <div class="d-flex justify-end gap-3 flex-wrap ">
          <VBtn class="me-2 mb-2" @click="openModalFormMasiveGlosa">
            <template #prepend>
              <VIcon start icon="tabler-folder" />
            </template>
            Glosa Masiva
          </VBtn>
          <VBtn @click="openModalFormServiceCreate()">
            Crear Servicio
          </VBtn>
          <VBtn @click="openModalListInvoicePayment()">
            Pagos
          </VBtn>
        </div>
      </VCardTitle>

      <VCardText>
        <FilterDialog ref="refFilterDialog" :options-filter="optionsFilter" @force-search="handleForceSearch"
          :table-loading="tableLoading">
        </FilterDialog>
      </VCardText>

      <VCardText class="mt-2">

        <TableFull v-model:selected="servicesIds" ref="refTableFull" :options="optionsTable"
          @update:loading="tableLoading = $event" @edit="openModalFormServiceEdit" @view="openModalFormServiceView">

          <template #item.actions2="{ item }">

            <VListItem @click="openModalListGlosa(item)">
              <template #prepend>
                <VIcon size="22" icon="tabler-square-rounded-arrow-right" />
              </template>
              <span>Listado Glosas</span>
            </VListItem>
          </template>

        </TableFull>
      </VCardText>
    </VCard>

    <ModalFormService ref="refModalFormService" @execute="handleForceSearch"></ModalFormService>

    <ModalListGlosa ref="refModalListGlosa"></ModalListGlosa>

    <ModalListInvoicePayment ref="refModalListInvoicePayment"></ModalListInvoicePayment>

    <ModalFormMasiveGlosa ref="refModalFormMasiveGlosa"></ModalFormMasiveGlosa>


  </div>
</template>
