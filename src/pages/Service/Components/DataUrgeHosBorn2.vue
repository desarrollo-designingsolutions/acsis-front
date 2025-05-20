<script setup lang="ts">
import { useAuthenticationStore } from "@/stores/useAuthenticationStore";
import { onMounted, ref } from 'vue';
import ModalFormHospitalization from "@/pages/Service/Components/ModalFormHospitalization.vue";
import ModalFormUrgency from "@/pages/Service/Components/ModalFormUrgency.vue";
import ModalFormNewlyBorn from "@/pages/Service/Components/ModalFormNewlyBorn.vue";

const props = defineProps<{
  invoice_id: string;
}>();

interface CountData {
  icon: string;
  color: string;
  title: string;
  value: string | number;
  secondary_data: string | number;
  change_label?: string;
  isHover: boolean;
  modal: string;
  type: string;
  hasServices: boolean;
  serviceId: string | null;
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
  to: {},
}));
const isLoading = ref<boolean>(false);
const hoverTimeout = ref<NodeJS.Timeout | null>(null);

// Modal refs
const refModalFormUrgency = ref();
const refModalFormHospitalization = ref();
const refModalFormNewlyBorn = ref();
const refModalQuestion = ref();

// Modal map
const serviceModalMap = {
  "SERVICE_TYPE_003": refModalFormUrgency,
  "SERVICE_TYPE_004": refModalFormHospitalization,
  "SERVICE_TYPE_005": refModalFormNewlyBorn,
};

// Methods
const updateCountData = (data: any): void => {
  countData.value = data.services.map((service: any) => ({
    ...service,
    isHover: false
  }));
};

const fetchData = async (): Promise<void> => {
  try {
    isLoading.value = true;
    const { data, response } = await useAxios(`/invoice/dataUrgeHosBorn/${props.invoice_id}`).get();

    if (response.status === 200 && data) {
      if (data !== false) {
        updateCountData(data);
      }
    }
  } catch (error) {
    console.error('Error fetching dashboard data:', error);
    // Mostrar notificación de error al usuario
  } finally {
    isLoading.value = false;
  }
};

const handleHover = (index: number, isEnter: boolean): void => {
  if (hoverTimeout.value) {
    clearTimeout(hoverTimeout.value);
  }

  hoverTimeout.value = setTimeout(() => {
    countData.value[index].isHover = isEnter;
  }, isEnter ? 50 : 200);
};

const openServiceModal = (type: string, params: Record<string, any> = {}, isViewMode: boolean = false) => {
  const modalRef = serviceModalMap[type]?.value;
  if (modalRef && typeof modalRef.openModal === 'function') {
    modalRef.openModal(params, isViewMode);
  } else {
    console.warn(`No modal found for service type: ${type}`);
  }
};

const openModalFormServiceCreate = (type: string) => {
  openServiceModal(type, { invoice_id: props.invoice_id });
};

const openModalFormServiceEdit = (data: CountData) => {
  if (data.serviceId) {
    openServiceModal(data.type, { serviceId: data.serviceId, invoice_id: props.invoice_id });
  }
};
const openModalFormServiceView = (data: CountData) => {
  if (data.serviceId) {
    openServiceModal(data.type, { serviceId: data.serviceId, invoice_id: props.invoice_id }, true);
  }
};

const confirmDelete = async (data: CountData) => {
  if (data.serviceId) {
    refModalQuestion.value.openModal(data.serviceId);
    refModalQuestion.value.componentData.title = `Eliminar ${data.title}`
    refModalQuestion.value.componentData.subTitle = `¿Estás seguro de eliminar el servicio de ${data.title}? Esta acción no se puede deshacer.`
  }
};

const handleDelete = async (id: string) => {
  try {
    const { response } = await useAxios(`/service/delete/${id}`).delete();
    if (response.status === 200) {
      // Mostrar notificación de éxito
      await fetchData();
    }
  } catch (error) {
    console.error('Error deleting service:', error);
    // Mostrar notificación de error
  }
};

// Handle success event from modals
const handleModalCreated = async () => {
  await fetchData();
};

// Lifecycle
onMounted(() => {
  fetchData();
});

// Cleanup
onUnmounted(() => {
  if (hoverTimeout.value) {
    clearTimeout(hoverTimeout.value);
  }
});
</script>

<template>
  <div class="service-dashboard">
    <!-- Service Cards Grid -->
    <VRow class="service-grid">
      <VCol v-for="(data, index) in countData" :key="index" cols="12" md="4" sm="6" class="service-col">
        <VCard class="service-card" :elevation="data.isHover ? 6 : 2" @mouseenter="handleHover(index, true)"
          @mouseleave="handleHover(index, false)" :style="{ '--card-color': data.color }" role="region"
          :aria-label="`Servicio de ${data.title}`">
          <VSkeletonLoader v-if="isLoading" type="card" class="h-100" />

          <template v-else>
            <!-- Card Header (ahora más compacto) -->
            <div class="service-card-header">
              <div class="header-content">
                <VAvatar :color="data.color" size="40" class="service-icon" :aria-hidden="true">
                  <VIcon :icon="data.icon" size="20" />
                </VAvatar>

                <div class="header-text">
                  <h3 class="service-title">{{ data.title }}</h3>
                  <div class="service-subtitle">
                    {{ data.value }} {{ data.secondary_data ? `• ${data.secondary_data}` : '' }}
                  </div>
                </div>

                <VChip size="small" :color="data.hasServices ? 'success' : 'grey'" variant="outlined"
                  class="status-chip">
                  {{ data.hasServices ? '✓' : '✗' }}
                </VChip>
              </div>
            </div>

            <!-- Card Content (se expande al hover) -->
            <VCardText class="service-card-content" :class="{ 'expanded': data.isHover }">
              <div class="content-details" v-if="data.isHover">
                <div class="detail-row">
                  <span class="detail-label">Cantidad:</span>
                  <span class="detail-value">{{ data.value }}</span>
                </div>

                <div class="detail-row" v-if="data.secondary_data">
                  <span class="detail-label">Detalles:</span>
                  <span class="detail-value">{{ data.secondary_data }}</span>
                </div>
              </div>

              <!-- Action Buttons (solo visibles al expandirse) -->
              <div class="action-buttons" :class="{ 'visible': data.isHover }">
                <VBtn v-if="!data.hasServices" :color="data.color" variant="elevated" prepend-icon="mdi-plus" block
                  @click.stop="openModalFormServiceCreate(data.type)" class="action-btn"
                  :aria-label="`Crear servicio de ${data.title}`">
                  Crear
                </VBtn>

                <template v-else>
                  <VBtn color="warning" variant="outlined" prepend-icon="tabler-eye"
                    @click.stop="openModalFormServiceView(data)" class="action-btn"
                    :aria-label="`Visualizar servicio de ${data.title}`">
                    Visualizar
                  </VBtn>
                  <VBtn color="warning" variant="outlined" prepend-icon="tabler-pencil"
                    @click.stop="openModalFormServiceEdit(data)" class="action-btn"
                    :aria-label="`Editar servicio de ${data.title}`">
                    Editar
                  </VBtn>
                  <VBtn color="error" variant="outlined" prepend-icon="mdi-delete" @click.stop="confirmDelete(data)"
                    class="action-btn" :aria-label="`Eliminar servicio de ${data.title}`">
                    Eliminar
                  </VBtn>
                </template>
              </div>
            </VCardText>
          </template>
        </VCard>
      </VCol>
    </VRow>

    <!-- Modals -->
    <ModalFormUrgency ref="refModalFormUrgency" @created="handleModalCreated" />
    <ModalFormHospitalization ref="refModalFormHospitalization" @created="handleModalCreated" />
    <ModalFormNewlyBorn ref="refModalFormNewlyBorn" @created="handleModalCreated" />
    <ModalQuestion ref="refModalQuestion" @success="handleDelete" />
  </div>
</template>

<style scoped lang="scss">
.service-dashboard {
  --card-hover-transition: all 0.3s cubic-bezier(0.25, 0.8, 0.25, 1);
  --card-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  --card-shadow-hover: 0 8px 16px rgba(0, 0, 0, 0.15);
  --card-border-radius: 12px;
  --card-padding: 1rem;
  --color-opacity: 0.08;
  --card-min-height: 80px;
  --card-expanded-height: 180px;
}

.service-card {
  position: relative;
  overflow: hidden;
  transition: var(--card-hover-transition);
  border-radius: var(--card-border-radius);
  min-height: var(--card-min-height);
  display: flex;
  flex-direction: column;
  background-color: rgb(var(--v-theme-surface));
  cursor: pointer;

  &:hover {
    transform: translateY(-4px);
    box-shadow: var(--card-shadow-hover) !important;

    .service-card-content {
      max-height: var(--card-expanded-height);
      opacity: 1;
      padding: var(--card-padding);
    }
  }
}

.service-card-header {
  padding: 0.75rem var(--card-padding);
  background-color: rgba(var(--v-theme-on-surface), var(--color-opacity));
  border-left: 4px solid rgb(var(--card-color));
  transition: all 0.3s ease;
}

.header-content {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.service-icon {
  transition: transform 0.3s ease;
  flex-shrink: 0;
}

.header-text {
  flex: 1;
  min-width: 0;
  /* Permite que el texto se trunque */
}

.service-title {
  font-size: 1rem;
  font-weight: 600;
  margin-bottom: 0.1rem;
  color: rgb(var(--v-theme-on-surface));
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.service-subtitle {
  font-size: 0.75rem;
  color: rgba(var(--v-theme-on-surface), 0.7);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.status-chip {
  font-weight: 500;
  margin-left: auto;
  flex-shrink: 0;
}

.service-card-content {
  max-height: 0;
  opacity: 0;
  padding: 0 var(--card-padding);
  overflow: hidden;
  transition: all 0.3s ease;
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.content-details {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  padding: 0.5rem 0;
}

.detail-row {
  display: flex;
  align-items: center;
  font-size: 0.875rem;
}

.detail-label {
  font-weight: 500;
  color: rgba(var(--v-theme-on-surface), 0.8);
  min-width: 80px;
}

.detail-value {
  color: rgb(var(--v-theme-on-surface));
}

.action-buttons {
  display: flex;
  gap: 0.5rem;
  margin-top: auto;
  opacity: 0;
  transition: opacity 0.2s ease 0.1s;

  .service-card:hover & {
    opacity: 1;
  }
}

.action-btn {
  flex: 1;
}

/* For touch devices */
@media (hover: none) {
  .service-card {
    min-height: auto;

    .service-card-content {
      max-height: none !important;
      opacity: 1 !important;
      padding: var(--card-padding) !important;
    }

    .action-buttons {
      opacity: 1 !important;
    }
  }
}

/* Responsive adjustments */
@media (max-width: 960px) {
  .service-col {
    flex: 0 0 50%;
    max-width: 50%;
  }
}

@media (max-width: 600px) {
  .service-col {
    flex: 0 0 100%;
    max-width: 100%;
  }

  .service-card-header {
    padding: 0.5rem;
  }

  .service-title {
    font-size: 0.9rem;
  }

  .action-buttons {
    flex-direction: column;
  }
}
</style>
