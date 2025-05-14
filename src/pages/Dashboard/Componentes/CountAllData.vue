<script setup lang="ts">
import { useAuthenticationStore } from "@/stores/useAuthenticationStore";
import { ref } from "vue";
const authenticationStore = useAuthenticationStore();

interface CountData {
  icon: string;
  color: string;
  title: string;
  value: string | number;
  secondary_data: string | number;
  change_label?: string;
  isHover: boolean;
  to: any;
}

const countData = ref<CountData[]>([
  {
    icon: '',
    color: '',
    title: '',
    value: 0,
    secondary_data: 0,
    change_label: undefined,
    isHover: false,
    to: {},
  },
  {
    icon: '',
    color: '',
    title: '',
    value: 0,
    secondary_data: 0,
    change_label: undefined,
    isHover: false,
    to: {},
  },
  {
    icon: '',
    color: '',
    title: '',
    value: 0,
    secondary_data: 0,
    change_label: undefined,
    isHover: false,
    to: {},
  },
  {
    icon: '',
    color: '',
    title: '',
    value: 0,
    secondary_data: 0,
    change_label: undefined,
    isHover: false,
    to: {},
  },
]);
const isLoading = ref<boolean>(false);
const startDate = ref<string>("2025-01-01"); // Valor por defecto
const endDate = ref<string>("2025-12-31"); // Valor por defecto

const fetchData = async () => {
  isLoading.value = true;
  const { data, response } = await useAxios(`/dashboard/countAllData`).get({
    params: {
      company_id: authenticationStore.company.id,
      start_date: startDate.value,
      end_date: endDate.value,
    }
  });

  isLoading.value = false;

  if (response.status == 200 && data) {

    countData.value[0] = data.invoiceCountData
    countData.value[1] = data.approvedVsGlosaData
    countData.value[2] = data.inReviewVsPendingData
    countData.value[3] = data.pendingPaymentsData
    countData.value[4] = data.averageResponseTimeData
    countData.value[5] = data.recoveredGlosasData

  }
};

// Llamar al cargar y al cambiar las fechas
onMounted(() => {
  fetchData();
});
</script>

<template>
  <VRow>
    <!-- Formulario de filtros -->
    <VCol cols="12" class="mb-4">
      <VRow>
        <VCol cols="12" md="4">
          <VTextField clearable v-model="startDate" label="Fecha de inicio" type="date" variant="outlined"
            :max="endDate" />
        </VCol>
        <VCol cols="12" md="4">
          <VTextField clearable v-model="endDate" label="Fecha de fin" type="date" variant="outlined"
            :min="startDate" />
        </VCol>
        <VCol cols="12" md="4" class="d-flex align-center">
          <VBtn color="primary" @click="fetchData" :loading="isLoading">
            Buscar
          </VBtn>
        </VCol>
      </VRow>
    </VCol>

    <!-- Tarjeta de datos -->
    <VCol v-for="(data, index) in countData" :key="index" cols="12" md="4" sm="6">
      <div>
        <VCard :to="data.to"
          :style="data.isHover ? `border-block-end-color: rgb(var(--v-theme-${data.color}))` : `border-block-end-color: rgba(var(--v-theme-${data.color}),0.38)`"
          @mouseenter="data.isHover = true" @mouseleave="data.isHover = false">
          <VSkeletonLoader :loading="isLoading" type="avatar,list-item, list-item">
            <VCardText>
              <div class="mb-2">
                <div class="text-body-1 font-weight-medium">{{ data.title }}</div>
              </div>
              <div class="d-flex align-center gap-x-2 mb-1">
                <VAvatar variant="tonal" :color="data.color" rounded size="small">
                  <VIcon :icon="data.icon" size="20" />
                </VAvatar>
                <h4 class="text-h4 font-weight-bold">
                  {{ data.value }}
                </h4>
              </div>
              <div v-if="data.change_label" class="text-body-2 text-error mb-1">
                {{ data.change_label }}
              </div>
              <div class="text-body-2">
                {{ data.secondary_data }}
              </div>
            </VCardText>
          </VSkeletonLoader>
        </VCard>
      </div>
    </VCol>

  </VRow>
</template>
