<script setup lang="ts">
import ModalFormMasiveGlosa from "@/pages/Glosa/Components/ModalFormMasive.vue";
import ModalListGlosa from "@/pages/Glosa/Components/ModalList.vue";
import ModalFormOtherService from "@/pages/Service/Components/ModalFormOtherService.vue";
import { useAuthenticationStore } from "@/stores/useAuthenticationStore";

const { toast } = useToast();

const props = defineProps<{
  invoice_id: string
}>()

const loading = reactive({
  btnCreate: false,
  excel: false,
})

const invoice_id = props.invoice_id;
const authenticationStore = useAuthenticationStore();
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
    { key: 'codigo_servicio', title: 'Código' },
    { key: 'nombre_servicio', title: 'Descripcion' },
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

const typeServiceEnumValues = ref<Array<{
  type: string,
  name: string,
}>>([]);

const fetchDataBtn = async () => {
  loading.btnCreate = true

  const { data, response } = await useAxios("/service/loadBtnCreate").get()

  if (response.status == 200 && data) {
    typeServiceEnumValues.value = data.typeServiceEnumValues;
  }
  loading.btnCreate = false
}


//Modales se los tipos de servicios
const refModalFormOtherService = ref()

const openModalFormServiceCreate = (type: string) => {
  if (type == "SERVICE_TYPE_007") {
    refModalFormOtherService.value.openModal({ invoice_id: invoice_id })
  }
}

const openModalFormServiceEdit = async (data: any) => {
  if (data.type == "SERVICE_TYPE_007") {
    refModalFormOtherService.value.openModal({ serviceId: data.id, invoice_id: invoice_id })
  }
}

const openModalFormServiceView = async (data: any) => {
  if (data.type == "SERVICE_TYPE_007") {
    refModalFormOtherService.value.openModal({ serviceId: data.id, invoice_id: invoice_id }, true)
  }
}

onMounted(() => {
  fetchDataBtn()
})
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

          <VBtn color="primary" append-icon="tabler-chevron-down" :loading="loading.btnCreate"
            :disabled="loading.btnCreate">
            Crear Servicio
            <VMenu activator="parent">
              <VList>
                <VListItem v-for="(item, index) in typeServiceEnumValues" :key="index"
                  @click="openModalFormServiceCreate(item.type)">
                  {{ item.name }}
                </VListItem>
              </VList>
            </VMenu>
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


    <ModalFormOtherService ref="refModalFormOtherService" @execute="handleForceSearch"></ModalFormOtherService>

    <ModalListGlosa ref="refModalListGlosa"></ModalListGlosa>

    <ModalFormMasiveGlosa ref="refModalFormMasiveGlosa"></ModalFormMasiveGlosa>
  </div>
</template>
