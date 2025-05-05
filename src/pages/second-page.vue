<script setup lang="ts">

import ModalForm from "@/pages/ModalForm.vue";
import { useAuthenticationStore } from "@/stores/useAuthenticationStore";

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
    { key: 'code', title: 'Código' },
    { key: 'description', title: 'Descripcion' },
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


//ModalForm
const refModalForm = ref()

const openModalFormCreate = () => {
  refModalForm.value.openModal({ invoice_id: invoice_id })
}

const openModalFormEdit = async (data: any) => {
  refModalForm.value.openModal({ invoice_id: invoice_id, id: data.id })
}

const openModalFormView = async (data: any) => {
  refModalForm.value.openModal({ id: data.id }, true)
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

</script>

<template>
  <div>

    <VCard>
      <VCardTitle class="d-flex justify-space-between">
        <span>
          Servicios
        </span>

        <div class="d-flex justify-end gap-3 flex-wrap ">
          <VBtn @click="openModalFormCreate()">
            Crear Glosa
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
          @update:loading="tableLoading = $event" @edit="openModalFormEdit" @view="openModalFormView">


        </TableFull>
      </VCardText>
    </VCard>

    <ModalForm ref="refModalForm" @execute="handleForceSearch"></ModalForm>

  </div>
</template>
