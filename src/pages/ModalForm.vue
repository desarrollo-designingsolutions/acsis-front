<script setup lang="ts">
import { useToast } from '@/composables/useToast';
import IErrorsBack from "@/interfaces/Axios/IErrorsBack";
import { useAuthenticationStore } from "@/stores/useAuthenticationStore";
import type { VForm } from "vuetify/components/VForm";

const { toast } = useToast()
const authenticationStore = useAuthenticationStore();
const { company } = storeToRefs(authenticationStore)

const errorsBack = ref<IErrorsBack>({});
const refForm = ref<VForm>();
const emit = defineEmits(["execute"])

const titleModal = ref<string>("Servicio")
const isDialogVisible = ref<boolean>(false)
const disabledFiledsView = ref<boolean>(false)

const isLoading = ref<boolean>(false)

const form = ref({
  id: null as string | null,
  invoice_id: null as string | null,
  company_id: null as string | null,
  code: null as string | null,
  description: null as string | null,
  quantity: null as string | null,
  unit_value: null as number | string | null,
})

const handleClearForm = () => {
  for (const key in form.value) {
    form.value[key] = null;
  }
}

const handleDialogVisible = () => {
  isDialogVisible.value = !isDialogVisible.value;
  handleClearForm()
};

const openModal = async ({ id = null, invoice_id }: any | null, disabled: boolean = false) => {
  handleDialogVisible();

  disabledFiledsView.value = disabled
  form.value.invoice_id = invoice_id
  form.value.id = id

  fetchDataForm()
};

const fetchDataForm = async () => {

  const url = form.value.id ? `/service/${form.value.id}/edit` : `/service/create`

  isLoading.value = true
  const { data, response } = await useAxios(url).get();

  if (response.status == 200 && data) {
    if (data.form) {
      form.value = cloneObject(data.form)

      form.value.unit_value = currencyFormat(
        formatToCurrencyFormat(data.form.unit_value)
      );
      dataCalculate.real_unit_value = cloneObject(data.form.unit_value);

    }
  }
  isLoading.value = false
}


const submitForm = async () => {
  const validation = await refForm.value?.validate()
  if (validation?.valid) {
    const url = form.value.id ? `/service/update/${form.value.id}` : `/service/store`

    form.value.company_id = authenticationStore.company.id;

    console.log(form.value);
    console.log(dataCalculate);

    isLoading.value = true;
    const { data, response } = await useAxios(url).post({
      ...form.value,
      unit_value: dataCalculate.real_unit_value
    });

    if (response.status == 200 && data) {
      handleDialogVisible();
      emit('execute');
    }
    if (data.code === 422) errorsBack.value = data.errors ?? {};

    isLoading.value = false;
  }
  else {
    toast('Faltan Campos Por Diligenciar', '', 'danger')
  }
}

defineExpose({
  openModal
})

const disabledTotal = ref(false)

// Regla para que sea mayor que cero
const positiveValidator = (value: number | string) => {
  const num = typeof value === 'string' ? parseFloat(value) : value
  if (isNaN(num) || num <= 0) {
    return 'El valor debe ser mayor que cero'
  }
  return true
}


//FORMATO COMPONENTE MONEDA
const dataCalculate = reactive({
  real_unit_value: 0 as number,
})

const dataReal = (data: any, field: string) => {
  dataCalculate[field] = data
}

function parseEuropeanNumber(value: string): number {
  if (!value) return 0;
  // Quitar el punto de miles y reemplazar la coma decimal
  const cleaned = value.replace(/\./g, '').replace(',', '.');
  return parseFloat(cleaned);
}

const total_value = computed(() => {
  const unitValue = parseEuropeanNumber(form.value.unit_value);
  const quantity = parseInt(form.value.quantity || '0', 10);
  const total = unitValue * quantity;
  return total.toLocaleString('es-CO', {
    style: 'currency',
    currency: 'COP',
  });
});
</script>

<template>
  <div>
    <VDialog v-model="isDialogVisible" :overlay="false" max-width="50rem" transition="dialog-transition" persistent>
      <DialogCloseBtn @click="handleDialogVisible" />
      <VCard :loading="isLoading" :disabled="isLoading" class="w-100">
        <!-- Toolbar -->

        <div>
          <VToolbar color="primary">
            <VToolbarTitle>{{ titleModal }}</VToolbarTitle>
          </VToolbar>
        </div>

        <VCardText>
          <VForm ref="refForm" @submit.prevent="() => { }">
            <VRow>
              <VCol cols="12" md="8">
                <AppTextField :requiredField="true" :disabled="disabledFiledsView" :rules="[requiredValidator]"
                  v-model="form.code" label="Código" :error-messages="errorsBack.code" @input="errorsBack.code = ''"
                  clearable />
              </VCol>
              <VCol cols="12">
                <AppTextarea :requiredField="true" clearable :disabled="disabledFiledsView" label="Descripción"
                  :rules="[requiredValidator]" v-model="form.description" :error-messages="errorsBack.description"
                  @input="errorsBack.description = ''" />
              </VCol>
              <VCol cols="12" md="6">
                <AppTextField :requiredField="true" :disabled="disabledFiledsView" :rules="[requiredValidator]"
                  v-model="form.quantity" label="Cantidad" type="number" :error-messages="errorsBack.quantity"
                  @input="errorsBack.quantity = ''" clearable />
              </VCol>
              <VCol cols="12" md="6">
                <FormatCurrency :requiredField="true" :disabled="disabledFiledsView || disabledTotal"
                  label="Valor Unitario" :rules="[requiredValidator, positiveValidator]" v-model="form.unit_value"
                  @realValue="dataReal($event, 'real_unit_value')" :error-messages="errorsBack.unit_value"
                  @input="errorsBack.unit_value = ''" clearable />
              </VCol>
              <VCol cols="12" md="6">
                <span> Total: </span>
                {{ total_value }}
              </VCol>
            </VRow>
          </VForm>
        </VCardText>

        <VCardText class="d-flex justify-end gap-3 flex-wrap mt-5">
          <VBtn color="secondary" :disabled="isLoading" :loading="isLoading" @click="handleDialogVisible">Cancelar
          </VBtn>
          <VBtn v-if="!disabledFiledsView" :disabled="isLoading" :loading="isLoading" @click="submitForm()"
            color="primary">
            Guardar
          </VBtn>
        </VCardText>

      </VCard>
    </VDialog>

  </div>
</template>
