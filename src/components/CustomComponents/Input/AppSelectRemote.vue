<script setup lang="ts">
import { computed, onMounted, ref, watch } from 'vue';

const props = defineProps({
  url: {
    type: String,
    required: true
  },
  modelValue: {
    type: [Array, Object, String, Number],
    default: () => []
  },
  params: {
    type: [Object],
    default: () => { }
  },
  multiple: {
    type: Boolean,
    default: false
  },
  itemTitle: {
    type: String,
    default: 'title'
  },
  itemValue: {
    type: String,
    default: 'value'
  },
  searchParam: {
    type: String,
    default: 'searchQueryInfinite'
  },
  arrayInfo: {
    type: String,
    required: true,
  },
  show: {
    type: Boolean,
    required: false,
  },
});

const emit = defineEmits(['update:modelValue']);

// Valor computado para manejar la doble vía
const localValue = computed({
  get: () => props.modelValue,
  set: (value) => emit('update:modelValue', value)
});

// Estado reactivo
const items = ref<any[]>([]);
const search = ref('');
const page = ref(1);
const loading = ref(false);
const hasMore = ref(true);
const error = ref('');


// Debounce para la búsqueda
let searchTimeout: ReturnType<typeof setTimeout>;

// Función principal para cargar datos
const loadItems = async (newSearch = false) => {
  error.value = '';
  if (loading.value) return;
  // if (loading.value || !hasMore.value) return;

  loading.value = true;

  try {
    if (newSearch) {
      page.value = 1;
      items.value = [];
      hasMore.value = true;
    }

    const { data } = await useAxios(props.url).post({
      [props.searchParam]: search.value,
      page: page.value,
      ...props.params,
    })

    const arrayInfo = props.arrayInfo + "_arrayInfo";
    const countLinks = props.arrayInfo + "_countLinks";

    if (data.data) { // Si usa paginación estilo Laravel
      items.value = newSearch ? data.data : [...items.value, ...data.data];
      hasMore.value = data.current_page < data.last_page;
    } else { // Si es un endpoint personalizado
      items.value = newSearch ? data[arrayInfo] : [...items.value, ...data[arrayInfo]];
      hasMore.value = data[countLinks].length > 1;
    }

    page.value++;
  } catch (err) {
    error.value = 'Error al buscar datos.' + err;
  } finally {
    loading.value = false;
  }
};


// Búsqueda con debounce
watch(search, (newVal) => {
  clearTimeout(searchTimeout);
  searchTimeout = setTimeout(() => loadItems(true), 500);
});

// Cargar datos iniciales
onMounted(() => {
  if (!props.show) {
    loadItems();
  }
});


// Mensaje para el slot no-data basado en el valor de localValue
const noDataMessage = computed(() => {
  if (loading.value) {
    return 'Buscando información...';
  }
  if (error.value) {
    return error.value;
  }

  return isEmpty(search.value)
    ? 'Escribe algo para buscar...'
    : `No se encontraron resultados para "<strong>${search.value}</strong>".`;
});

const clearText = () => {
  search.value = '';
}
</script>

<template>
  <AppSelect @blur="clearText" returnObject v-model="localValue" :items="items" :item-title="itemTitle"
    :item-value="itemValue" :multiple="multiple" :search="search" :loading="loading"
    @update:search="(value) => search = value" clearable>
    <template #prepend-item class="sticky-search">
      <VListItem>
        <AppTextField v-model="search" placeholder="Teclee para buscar..." variant="outlined" density="compact"
          hide-details clearable @click:clear="loadItems(true)" />
      </VListItem>
    </template>
    <!-- Codigo que itera sobre las ranuras disponibles en el componente padre e individualmente rinde cada ranura con sus propias propiedades -->
    <template v-for="(_, name) in $slots" #[name]="slotProps">
      <slot :name="name" v-bind="slotProps || {}" />
    </template>
    <!-- Codigo que itera sobre las ranuras disponibles en el componente padre e individualmente rinde cada ranura con sus propias propiedades -->


    <template v-slot:no-data>
      <VListItem>
        <VListItemTitle class="text-center" v-html="noDataMessage"></VListItemTitle>
      </VListItem>
    </template>
  </AppSelect>
</template>

<style scoped>
:deep(.sticky-search) {
  position: sticky;
  z-index: 999;

  /* Asegúrate de que esté encima de otros elementos */
  background: white;

  /* Asegura que ocupe todo el ancho del contenedor */
  box-shadow: 0 2px 10px rgba(0, 0, 0, 10%);
  inline-size: 100%;
  inset-block-start: 0;

  /* Color de fondo según tu tema */
  padding-block: 8px;

  /* Agrega sombra para hacerlo más visible */
}

/* Para temas oscuros */
:deep(.v-theme--dark .sticky-search) {
  background: #1e1e1e;

  /* Color de fondo oscuro */
  color: white;

  /* Asegura que el texto sea visible en tema oscuro */
}
</style>
