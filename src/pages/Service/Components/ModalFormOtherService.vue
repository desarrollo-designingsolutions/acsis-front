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
const disabledTotal = ref(false)
const tipoOtrosServicios_arrayInfo = ref([])
const conceptoRecaudo_arrayInfo = ref([])
const service_id = ref<null | string>(null)

const form = ref({
  id: null as string | null,
  invoice_id: null as string | null,
  numAutorizacion: null as string | null,
  idMIPRES: null as string | null,
  fechaSuministroTecnologia: null as string | null,
  tipoOS_id: null as string | null,
  codTecnologiaSalud: null as string | null,
  nomTecnologiaSalud: null as string | null,
  cantidadOS: null as string | null,
  vrUnitOS: null as string | null,
  valorPagoModerador: null as string | null,
  vrServicio: null as string | null,
  conceptoRecaudo_id: null as string | null,
})

const dataCalculate = reactive({
  real_vrUnitOS: 0 as number,
  real_valorPagoModerador: 0 as number,
  real_vrServicio: 0 as number,
})

const handleClearForm = () => {
  for (const key in form.value) {
    form.value[key] = null
  }
}

const handleDialogVisible = () => {
  isDialogVisible.value = !isDialogVisible.value;
  if (!isDialogVisible.value) {
    handleClearForm();
  }
};

const openModal = async ({ serviceId = null, invoice_id }: any | null, disabled: boolean = false) => {
  handleDialogVisible();
  disabledFiledsView.value = disabled;
  form.value.invoice_id = invoice_id;
  service_id.value = serviceId;

  await fetchDataForm();
};

const fetchDataForm = async () => {
  try {
    isLoading.value = true;
    const url = service_id.value ? `/service/otherService/${service_id.value}/edit` : `/service/otherService/create`;
    const { data, response } = await useAxios(url).get();

    if (response.status === 200 && data) {

      tipoOtrosServicios_arrayInfo.value = data.tipoOtrosServicios_arrayInfo
      conceptoRecaudo_arrayInfo.value = data.conceptoRecaudo_arrayInfo

      if (data.form) {
        form.value = cloneObject(data.form);

        form.value.vrUnitOS = currencyFormat(formatToCurrencyFormat(data.form.vrUnitOS));
        dataCalculate.real_vrUnitOS = cloneObject(data.form.vrUnitOS);
        form.value.valorPagoModerador = currencyFormat(formatToCurrencyFormat(data.form.valorPagoModerador));
        dataCalculate.real_valorPagoModerador = cloneObject(data.form.valorPagoModerador);
        form.value.vrServicio = currencyFormat(formatToCurrencyFormat(data.form.vrServicio));
        dataCalculate.real_vrServicio = cloneObject(data.form.vrServicio);
      }
    }
  } catch (error) {
    toast('Error al cargar los datos', '', 'warning');
  } finally {
    isLoading.value = false;
  }
};

const submitForm = async () => {
  try {
    const validation = await refForm.value?.validate();
    if (!validation?.valid) {
      toast('Faltan campos por diligenciar', '', 'warning');
      return;
    }

    isLoading.value = true;
    const url = form.value.id ? `/service/otherService/update/${form.value.id}` : `/service/otherService/store`;

    const payload = {
      ...form.value,
      service_id: service_id.value,
      company_id: authenticationStore.company.id,
      vrUnitOS: dataCalculate.real_vrUnitOS,
      valorPagoModerador: dataCalculate.real_valorPagoModerador,
      vrServicio: dataCalculate.real_vrServicio,
    };

    const { data, response } = await useAxios(url).post(payload);

    if (response.status === 200 && data.code == 200) {
      handleDialogVisible();
      emit('execute');
    }

    if (data.code === 422) {
      errorsBack.value = data.errors ?? {};
    }
  } catch (error) {
    toast('Error al guardar el servicio', '', 'warning');
  } finally {
    isLoading.value = false;
  }
};

// Validators
const positiveValidator = (value: number | string) => {
  const num = typeof value === 'string' ? parseFloat(value) : value;
  return isNaN(num) || num <= 0 ? 'El valor debe ser mayor que cero' : true;
};

// Utility functions
const parseEuropeanNumber = (value: string): number => {
  if (!value) return 0;
  return parseFloat(value.replace(/\./g, '').replace(',', '.'));
};

const dataReal = (data: any, field: string) => {
  dataCalculate[field] = data;
};


defineExpose({
  openModal
});
</script>

<template>
  <div>
    <VDialog v-model="isDialogVisible" :overlay="false" max-width="65rem" transition="dialog-transition" persistent>
      <DialogCloseBtn @click="handleDialogVisible" />
      <VCard :loading="isLoading" :disabled="isLoading">

        <VToolbar color="primary" class="rounded-t">
          <VToolbarTitle>
            {{ titleModal }}
          </VToolbarTitle>
        </VToolbar>

        <VCardText class="pt-6">
          <VForm ref="refForm" @submit.prevent>
            <VRow>
              <VCol cols="12" md="6">
                <AppTextField clearable label="numAutorizacion" v-model="form.numAutorizacion" :requiredField="true"
                  :rules="[requiredValidator]" :error-messages="errorsBack.numAutorizacion"
                  @input="errorsBack.numAutorizacion = ''" :disabled="disabledFiledsView" />
              </VCol>
              <VCol cols="12" md="6">
                <AppTextField clearable label="idMIPRES" v-model="form.idMIPRES" :requiredField="true"
                  :rules="[requiredValidator]" :error-messages="errorsBack.idMIPRES" @input="errorsBack.idMIPRES = ''"
                  :disabled="disabledFiledsView" />
              </VCol>
              <VCol cols="12" md="6">
                <AppTextField clearable label="fechaSuministroTecnologia" v-model="form.fechaSuministroTecnologia"
                  :requiredField="true" :rules="[requiredValidator]"
                  :error-messages="errorsBack.fechaSuministroTecnologia"
                  @input="errorsBack.fechaSuministroTecnologia = ''" :disabled="disabledFiledsView" type="date" />
              </VCol>
              <VCol cols="12" md="6">
                <AppSelectRemote clearable label="tipoOS" v-model="form.tipoOS_id" :rules="[requiredValidator]"
                  :error-messages="errorsBack.tipoOS_id" @input="errorsBack.tipoOS_id = ''"
                  :disabled="disabledFiledsView" url="/selectInfiniteTipoOtrosServicios" array-info="tipoOtrosServicios"
                  :itemsData="tipoOtrosServicios_arrayInfo" :firstFetch="false" />
              </VCol>
              <VCol cols="12" md="6">
                <AppTextField clearable label="codTecnologiaSalud" v-model="form.codTecnologiaSalud"
                  :requiredField="true" :rules="[requiredValidator]" :error-messages="errorsBack.codTecnologiaSalud"
                  @input="errorsBack.codTecnologiaSalud = ''" :disabled="disabledFiledsView" />
              </VCol>
              <VCol cols="12" md="6">
                <AppTextField clearable label="nomTecnologiaSalud" v-model="form.nomTecnologiaSalud"
                  :requiredField="true" :rules="[requiredValidator]" :error-messages="errorsBack.nomTecnologiaSalud"
                  @input="errorsBack.nomTecnologiaSalud = ''" :disabled="disabledFiledsView" />
              </VCol>
              <VCol cols="12" md="6">
                <AppTextField clearable label="cantidadOS" v-model="form.cantidadOS" :requiredField="true"
                  :rules="[requiredValidator]" :error-messages="errorsBack.cantidadOS"
                  @input="errorsBack.cantidadOS = ''" :disabled="disabledFiledsView" />
              </VCol>
              <VCol cols="12" md="6">
                <FormatCurrency clearable label="vrUnitOS" v-model="form.vrUnitOS" :requiredField="true"
                  :rules="[requiredValidator, positiveValidator]" :error-messages="errorsBack.vrUnitOS"
                  @input="errorsBack.vrUnitOS = ''" :disabled="disabledFiledsView || disabledTotal"
                  @realValue="dataReal($event, 'real_vrUnitOS')" />
              </VCol>
              <VCol cols="12" md="6">
                <FormatCurrency clearable label="valorPagoModerador" v-model="form.valorPagoModerador"
                  :requiredField="true" :rules="[requiredValidator, positiveValidator]"
                  :error-messages="errorsBack.valorPagoModerador" @input="errorsBack.valorPagoModerador = ''"
                  :disabled="disabledFiledsView || disabledTotal"
                  @realValue="dataReal($event, 'real_valorPagoModerador')" />
              </VCol>
              <VCol cols="12" md="6">
                <FormatCurrency clearable label="vrServicio" v-model="form.vrServicio" :requiredField="true"
                  :rules="[requiredValidator, positiveValidator]" :error-messages="errorsBack.vrServicio"
                  @input="errorsBack.vrServicio = ''" :disabled="disabledFiledsView || disabledTotal"
                  @realValue="dataReal($event, 'real_vrServicio')" />
              </VCol>
              <VCol cols="12" md="6">
                <AppSelectRemote clearable label="conceptoRecaudo" v-model="form.conceptoRecaudo_id"
                  :rules="[requiredValidator]" :error-messages="errorsBack.conceptoRecaudo_id"
                  @input="errorsBack.conceptoRecaudo_id = ''" :disabled="disabledFiledsView"
                  url="/selectInfiniteConceptoRecaudo" array-info="conceptoRecaudo"
                  :itemsData="conceptoRecaudo_arrayInfo" :firstFetch="false" />
              </VCol>

            </VRow>
          </VForm>
        </VCardText>

        <VDivider />

        <VCardText class="d-flex justify-end gap-3 flex-wrap">
          <VSpacer />
          <VBtn color="secondary" :disabled="isLoading" @click="handleDialogVisible" class="me-3">
            Cancelar
          </VBtn>
          <VBtn v-if="!disabledFiledsView" color="primary" :loading="isLoading" :disabled="isLoading"
            @click="submitForm">
            Guardar
          </VBtn>
        </VCardText>
      </VCard>
    </VDialog>
  </div>
</template>
