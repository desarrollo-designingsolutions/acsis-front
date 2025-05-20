<script setup lang="ts">
import { useAuthenticationStore } from "@/stores/useAuthenticationStore";
import { onMounted, ref } from 'vue';

const props = defineProps<{
  invoice_id: string
}>()

interface CountData {
  icon: string;
  color: string;
  title: string;
  value: string | number;
  secondary_data: string | number;
  change_label?: string;
  isHover: boolean;
  to: Record<string, unknown>;
}

// Initialize store
const authenticationStore = useAuthenticationStore();

// State
const countData = ref<CountData[]>(Array(3).fill({
  icon: '',
  color: '',
  title: '',
  value: 0,
  secondary_data: 0,
  change_label: undefined,
  isHover: false,
}));

const isLoading = ref<boolean>(false);


// Methods
const updateCountData = (data: any): void => {
  countData.value = data.services;
};

const fetchData = async (): Promise<void> => {
  try {
    isLoading.value = true;
    const { data, response } = await useAxios(`/invoice/dataUrgeHosBorn/${props.invoice_id}`).get();
    if (response.status === 200 && data) {
      if (data != false) {
        updateCountData(data);

      }
    }
  } catch (error) {
    console.error('Error fetching dashboard data:', error);
    // Here you could add error handling, like showing a notification
  } finally {
    isLoading.value = false;
  }
};


// Lifecycle
onMounted(fetchData);
</script>

<template>
  <VRow>

    <!-- Stats Cards -->
    <VCol v-for="(data, index) in countData" :key="index" cols="12" md="4" sm="6">
      <VCard :style="`border-block-end-color: ${data.isHover
        ? `rgb(var(--v-theme-${data.color}))`
        : `rgba(var(--v-theme-${data.color}),0.38)`
        }`" @mouseenter="data.isHover = true" @mouseleave="data.isHover = false" class="h-100">
        <VSkeletonLoader :loading="isLoading" type="avatar,list-item,list-item">
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
    </VCol>
  </VRow>
</template>
