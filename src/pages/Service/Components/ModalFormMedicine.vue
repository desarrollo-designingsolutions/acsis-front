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

const cie10_arrayInfo = ref([])
const tipoMedicamentoPosVersion2_arrayInfo = ref([])
const umm_arrayInfo = ref([])
const conceptoRecaudo_arrayInfo = ref([])

const service_id = ref<null | string>(null)

const form = ref({
  id: null as string | null,
  invoice_id: null as string | null,

  numAutorizacion: null as string | null,
  idMIPRES: null as string | null,
  fechaDispensAdmon: null as string | null,
  codDiagnosticoPrincipal_id: null as string | null,
  codDiagnosticoRelacionado_id: null as string | null,
  tipoMedicamento_id: null as string | null,
  codTecnologiaSalud: null as string | null,
  nomTecnologiaSalud: null as string | null,
  concentracionMedicamento: null as string | null,
  unidadMedida_id: null as string | null,
  formaFarmaceutica: null as string | null,
  unidadMinDispensa: null as string | null,
  cantidadMedicamento: null as string | null,
  diasTratamiento: null as string | null,
  vrUnitMedicamento: null as string | null,
  valorPagoModerador: null as string | null,
  vrServicio: null as string | null,
  conceptoRecaudo_id: null as string | null,
})

const dataCalculate = reactive({
  real_vrUnitMedicamento: 0 as number,
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
    const url = service_id.value ? `/service/medicine/${service_id.value}/edit` : `/service/medicine/create`;
    const { data, response } = await useAxios(url).get();

    if (response.status === 200 && data) {

      cie10_arrayInfo.value = data.cie10_arrayInfo
      tipoMedicamentoPosVersion2_arrayInfo.value = data.tipoMedicamentoPosVersion2_arrayInfo
      umm_arrayInfo.value = data.umm_arrayInfo
      conceptoRecaudo_arrayInfo.value = data.conceptoRecaudo_arrayInfo


      if (data.form) {
        form.value = cloneObject(data.form);

        form.value.vrUnitMedicamento = currencyFormat(formatToCurrencyFormat(data.form.vrUnitMedicamento));
        dataCalculate.real_vrUnitMedicamento = cloneObject(data.form.vrUnitMedicamento);
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
    const url = form.value.id ? `/service/medicine/update/${form.value.id}` : `/service/medicine/store`;

    const payload = {
      ...form.value,
      service_id: service_id.value,
      company_id: authenticationStore.company.id,
      vrUnitMedicamento: dataCalculate.real_vrUnitMedicamento,
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
                <AppTextField clearable label="fechaDispensAdmon" v-model="form.fechaDispensAdmon" :requiredField="true"
                  :rules="[requiredValidator]" :error-messages="errorsBack.fechaDispensAdmon"
                  @input="errorsBack.fechaDispensAdmon = ''" :disabled="disabledFiledsView" type="date" />
              </VCol>

              <VCol cols="12" md="6">
                <AppSelectRemote clearable label="codDiagnosticoPrincipal" v-model="form.codDiagnosticoPrincipal_id"
                  :requiredField="true" :rules="[requiredValidator]"
                  :error-messages="errorsBack.codDiagnosticoPrincipal_id"
                  @input="errorsBack.codDiagnosticoPrincipal_id = ''" :disabled="disabledFiledsView"
                  url="/selectInfiniteCie10" array-info="cie10" :itemsData="cie10_arrayInfo" :firstFetch="false" />
              </VCol>

              <VCol cols="12" md="6">
                <AppSelectRemote clearable label="codDiagnosticoRelacionado" v-model="form.codDiagnosticoRelacionado_id"
                  :requiredField="true" :rules="[requiredValidator]"
                  :error-messages="errorsBack.codDiagnosticoRelacionado_id"
                  @input="errorsBack.codDiagnosticoRelacionado_id = ''" :disabled="disabledFiledsView"
                  url="/selectInfiniteCie10" array-info="cie10" :itemsData="cie10_arrayInfo" :firstFetch="false" />
              </VCol>

              <VCol cols="12" md="6">
                <AppSelectRemote clearable label="tipoMedicamento" v-model="form.tipoMedicamento_id"
                  :requiredField="true" :rules="[requiredValidator]" :error-messages="errorsBack.tipoMedicamento_id"
                  @input="errorsBack.tipoMedicamento_id = ''" :disabled="disabledFiledsView"
                  url="/selectInfiniteTipoMedicamentoPosVersion2" array-info="tipoMedicamentoPosVersion2"
                  :itemsData="tipoMedicamentoPosVersion2_arrayInfo" :firstFetch="false" />
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
                <AppTextField clearable label="concentracionMedicamento" v-model="form.concentracionMedicamento"
                  :requiredField="true" :rules="[requiredValidator]"
                  :error-messages="errorsBack.concentracionMedicamento"
                  @input="errorsBack.concentracionMedicamento = ''" :disabled="disabledFiledsView" />
              </VCol>

              <VCol cols="12" md="6">
                <AppSelectRemote clearable label="unidadMedida" v-model="form.unidadMedida_id" :requiredField="true"
                  :rules="[requiredValidator]" :error-messages="errorsBack.unidadMedida_id"
                  @input="errorsBack.unidadMedida_id = ''" :disabled="disabledFiledsView" url="/selectInfiniteUmm"
                  array-info="umm" :itemsData="umm_arrayInfo" :firstFetch="false" />
              </VCol>

              <VCol cols="12" md="6">
                <AppTextField clearable label="formaFarmaceutica" v-model="form.formaFarmaceutica" :requiredField="true"
                  :rules="[requiredValidator]" :error-messages="errorsBack.formaFarmaceutica"
                  @input="errorsBack.formaFarmaceutica = ''" :disabled="disabledFiledsView" />
              </VCol>

              <VCol cols="12" md="6">
                <AppTextField clearable label="unidadMinDispensa" v-model="form.unidadMinDispensa" :requiredField="true"
                  :rules="[requiredValidator]" :error-messages="errorsBack.unidadMinDispensa"
                  @input="errorsBack.unidadMinDispensa = ''" :disabled="disabledFiledsView" />
              </VCol>

              <VCol cols="12" md="6">
                <AppTextField clearable label="cantidadMedicamento" v-model="form.cantidadMedicamento"
                  :requiredField="true" :rules="[requiredValidator]" :error-messages="errorsBack.cantidadMedicamento"
                  @input="errorsBack.cantidadMedicamento = ''" :disabled="disabledFiledsView" />
              </VCol>

              <VCol cols="12" md="6">
                <AppTextField clearable label="diasTratamiento" v-model="form.diasTratamiento" :requiredField="true"
                  :rules="[requiredValidator]" :error-messages="errorsBack.diasTratamiento"
                  @input="errorsBack.diasTratamiento = ''" :disabled="disabledFiledsView" />
              </VCol>

              <VCol cols="12" md="6">
                <FormatCurrency clearable label="vrUnitMedicamento" v-model="form.vrUnitMedicamento"
                  :requiredField="true" :rules="[requiredValidator, positiveValidator]"
                  :error-messages="errorsBack.vrUnitMedicamento" @input="errorsBack.vrUnitMedicamento = ''"
                  :disabled="disabledFiledsView" @realValue="dataReal($event, 'real_vrUnitMedicamento')" />
              </VCol>

              <VCol cols="12" md="6">
                <FormatCurrency clearable label="valorPagoModerador" v-model="form.valorPagoModerador"
                  :requiredField="true" :rules="[requiredValidator, positiveValidator]"
                  :error-messages="errorsBack.valorPagoModerador" @input="errorsBack.valorPagoModerador = ''"
                  :disabled="disabledFiledsView" @realValue="dataReal($event, 'real_valorPagoModerador')" />
              </VCol>

              <VCol cols="12" md="6">
                <FormatCurrency clearable label="vrServicio" v-model="form.vrServicio" :requiredField="true"
                  :rules="[requiredValidator, positiveValidator]" :error-messages="errorsBack.vrServicio"
                  @input="errorsBack.vrServicio = ''" :disabled="disabledFiledsView"
                  @realValue="dataReal($event, 'real_vrServicio')" />
              </VCol>

              <VCol cols="12" md="6">
                <AppSelectRemote clearable label="conceptoRecaudo" v-model="form.conceptoRecaudo_id"
                  :requiredField="true" :rules="[requiredValidator]" :error-messages="errorsBack.conceptoRecaudo_id"
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
