<script setup lang="ts">

// Interfaz para tipar los datos
interface ValidationError {
  level: string;
  key: string;
  data: string;
  message: string;
}

interface ModalData {
  isValid: boolean;
  message: string | null;
  errors: ValidationError[];
}

const emit = defineEmits(["continue", "cancel"]);

const titleModal = ref<string>("Errores en la validación");
const isDialogVisible = ref<boolean>(false);
const loading = reactive({
  excel: false,
  getData: false,
});

// Inicializar objData con un valor por defecto
const objData = ref<ModalData>({ isValid: false, message: "", errors: [] });

const handleDialogVisible = () => {
  isDialogVisible.value = !isDialogVisible.value;
  if (!isDialogVisible.value) {
    objData.value = { isValid: false, message: "", errors: [] }; // Limpiar datos al cerrar
  }
};

const openModal = async (data: any) => {
  handleDialogVisible();
  objData.value = data;
};

defineExpose({ openModal });


// Headers para la tabla
const inputsTableFilter = [
  { title: "Nivel", key: "level" },
  { title: "Elemento", key: "key" },
  { title: "Dato", key: "data" },
  { title: "Descripción error", key: "message" },
];

// Opciones reactivas para la tabla
const options = ref({
  page: 1,
  itemsPerPage: 10,
  sortBy: [],
  sortDesc: [],
});
const search = ref("");

const downloadExcel = async () => {
  exportArrayToExcel(objData.value.errors, "Lista de errores")
};

const cancel = () => {
  emit("cancel");
  handleDialogVisible();
};

</script>

<template>
  <div>
    <VDialog v-model="isDialogVisible" :overlay="false" transition="dialog-transition" persistent>
      <DialogCloseBtn @click="handleDialogVisible" />
      <VCard :loading="loading.getData" :disabled="loading.getData" class="w-100">
        <div>
          <VToolbar color="primary">
            <VToolbarTitle>{{ titleModal }}</VToolbarTitle>
          </VToolbar>
        </div>

        <VCardText>
          <VRow>
            <VCol cols="12" sm="6">
              <AppTextField v-model="search" density="compact" placeholder="Buscar..." append-inner-icon="tabler-search"
                single-line hide-details clearable />
            </VCol>
            <VCol cols="12" sm="6" class="d-flex justify-end">
              <VBtn :loading="loading.excel" :disabled="loading.excel" @click="downloadExcel">
                Exportar a Excel
              </VBtn>
            </VCol>
          </VRow>
        </VCardText>

        <VCardText>
          <VDataTable v-model:options="options" v-model:search="search" :headers="inputsTableFilter"
            :items="objData.errors" :items-per-page="options.itemsPerPage" :page="options.page" />
        </VCardText>

        <VCardText class="d-flex justify-end gap-3 flex-wrap">
          <VBtn :loading="loading.getData" color="secondary" variant="tonal" @click="cancel">
            Cerrar
          </VBtn>
        </VCardText>
      </VCard>
    </VDialog>
  </div>
</template>
