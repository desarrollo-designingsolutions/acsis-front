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

const titleModal = ref<string>("Servicio: Procedimientos")
const isDialogVisible = ref<boolean>(false)
const disabledFiledsView = ref<boolean>(false)
const isLoading = ref<boolean>(false)

const cupsRips_arrayInfo = ref([])
const viaIngresoUsuario_arrayInfo = ref([])
const modalidadAtencion_arrayInfo = ref([])
const grupoServicio_arrayInfo = ref([])
const servicio_arrayInfo = ref([])
const ripsFinalidadConsultaVersion2_arrayInfo = ref([])
const cie10_arrayInfo = ref([])
const conceptoRecaudo_arrayInfo = ref([])

const service_id = ref<null | string>(null)

const form = ref({
  id: null as string | null,
  invoice_id: null as string | null,

  fechaInicioAtencion: null as string | null,
  idMIPRES: null as string | null,
  numAutorizacion: null as string | null,
  codProcedimiento_id: null as string | null,
  viaIngresoServicioSalud_id: null as string | null,
  modalidadGrupoServicioTecSal_id: null as string | null,
  grupoServicios_id: null as string | null,
  codServicio_id: null as string | null,
  finalidadTecnologiaSalud_id: null as string | null,
  codDiagnosticoPrincipal_id: null as string | null,
  codDiagnosticoRelacionado_id: null as string | null,
  codComplicacion_id: null as string | null,
  valorPagoModerador: null as string | null,
  vrServicio: null as string | null,
  conceptoRecaudo_id: null as string | null,
})

const dataCalculate = reactive({
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
    const url = service_id.value ? `/service/procedure/${service_id.value}/edit` : `/service/procedure/create`;
    const { data, response } = await useAxios(url).get();

    if (response.status === 200 && data) {

      cupsRips_arrayInfo.value = data.cupsRips_arrayInfo
      viaIngresoUsuario_arrayInfo.value = data.viaIngresoUsuario_arrayInfo
      modalidadAtencion_arrayInfo.value = data.modalidadAtencion_arrayInfo
      grupoServicio_arrayInfo.value = data.grupoServicio_arrayInfo
      servicio_arrayInfo.value = data.servicio_arrayInfo
      ripsFinalidadConsultaVersion2_arrayInfo.value = data.ripsFinalidadConsultaVersion2_arrayInfo
      cie10_arrayInfo.value = data.cie10_arrayInfo
      conceptoRecaudo_arrayInfo.value = data.conceptoRecaudo_arrayInfo


      if (data.form) {
        form.value = cloneObject(data.form);

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
    const url = form.value.id ? `/service/procedure/update/${form.value.id}` : `/service/procedure/store`;

    const payload = {
      ...form.value,
      service_id: service_id.value,
      company_id: authenticationStore.company.id,
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
  return isNaN(num) || num < 0 ? 'El valor debe ser positivo' : true;
};

const lessThanVrService = (value: number | string) => {
  const vrService = dataCalculate.real_vrServicio;
  const num = typeof value === 'string' ? parseFloat(value) : value;
  return isNaN(num) || vrService < num ? 'El valor debe ser menor o igual a ' + vrService : true;
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
                <AppTextField clearable label="fechaInicioAtencion" v-model="form.fechaInicioAtencion"
                  :requiredField="true" :rules="[requiredValidator]" :error-messages="errorsBack.fechaInicioAtencion"
                  @input="errorsBack.fechaInicioAtencion = ''" :disabled="disabledFiledsView" type="date" />
              </VCol>

              <VCol cols="12" md="6">
                <AppTextField clearable label="idMIPRES" v-model="form.idMIPRES" :requiredField="true"
                  :rules="[requiredValidator]" :error-messages="errorsBack.idMIPRES" @input="errorsBack.idMIPRES = ''"
                  :disabled="disabledFiledsView" />
              </VCol>

              <VCol cols="12" md="6">
                <AppTextField clearable label="numAutorizacion" v-model="form.numAutorizacion" :requiredField="true"
                  :rules="[requiredValidator]" :error-messages="errorsBack.numAutorizacion"
                  @input="errorsBack.numAutorizacion = ''" :disabled="disabledFiledsView" />
              </VCol>

              <VCol cols="12" md="6">
                <AppSelectRemote clearable label="codProcedimiento" v-model="form.codProcedimiento_id"
                  :requiredField="true" :rules="[requiredValidator]" :error-messages="errorsBack.codProcedimiento_id"
                  @input="errorsBack.codProcedimiento_id = ''" :disabled="disabledFiledsView"
                  url="/selectInfiniteCupsRips" array-info="cupsRips" :itemsData="cupsRips_arrayInfo"
                  :firstFetch="false" />
              </VCol>

              <VCol cols="12" md="6">
                <AppSelectRemote clearable label="viaIngresoServicioSalud" v-model="form.viaIngresoServicioSalud_id"
                  :requiredField="true" :rules="[requiredValidator]"
                  :error-messages="errorsBack.viaIngresoServicioSalud_id"
                  @input="errorsBack.viaIngresoServicioSalud_id = ''" :disabled="disabledFiledsView"
                  url="/selectInfiniteViaIngresoUsuario" array-info="viaIngresoUsuario"
                  :itemsData="viaIngresoUsuario_arrayInfo" :firstFetch="false" />
              </VCol>

              <VCol cols="12" md="6">
                <AppSelectRemote clearable label="modalidadGrupoServicioTecSal"
                  v-model="form.modalidadGrupoServicioTecSal_id" :requiredField="true" :rules="[requiredValidator]"
                  :error-messages="errorsBack.modalidadGrupoServicioTecSal_id"
                  @input="errorsBack.modalidadGrupoServicioTecSal_id = ''" :disabled="disabledFiledsView"
                  url="/selectInfiniteModalidadAtencion" array-info="modalidadAtencion"
                  :itemsData="modalidadAtencion_arrayInfo" :firstFetch="false" />
              </VCol>

              <VCol cols="12" md="6">
                <AppSelectRemote clearable label="grupoServicios" v-model="form.grupoServicios_id" :requiredField="true"
                  :rules="[requiredValidator]" :error-messages="errorsBack.grupoServicios_id"
                  @input="errorsBack.grupoServicios_id = ''" :disabled="disabledFiledsView"
                  url="/selectInfiniteGrupoServicio" array-info="grupoServicio" :itemsData="grupoServicio_arrayInfo"
                  :firstFetch="false" />
              </VCol>

              <VCol cols="12" md="6">
                <AppSelectRemote clearable label="codServicio" v-model="form.codServicio_id" :requiredField="true"
                  :rules="[requiredValidator]" :error-messages="errorsBack.codServicio_id"
                  @input="errorsBack.codServicio_id = ''" :disabled="disabledFiledsView" url="/selectInfiniteServicio"
                  array-info="servicio" :itemsData="servicio_arrayInfo" :firstFetch="false" />
              </VCol>

              <VCol cols="12" md="6">
                <AppSelectRemote clearable label="finalidadTecnologiaSalud" v-model="form.finalidadTecnologiaSalud_id"
                  :requiredField="true" :rules="[requiredValidator]"
                  :error-messages="errorsBack.finalidadTecnologiaSalud_id"
                  @input="errorsBack.finalidadTecnologiaSalud_id = ''" :disabled="disabledFiledsView"
                  url="/selectInfiniteRipsFinalidadConsultaVersion2" array-info="ripsFinalidadConsultaVersion2"
                  :itemsData="ripsFinalidadConsultaVersion2_arrayInfo" :firstFetch="false" />
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
                  :error-messages="errorsBack.codDiagnosticoRelacionado_id"
                  @input="errorsBack.codDiagnosticoRelacionado_id = ''" :disabled="disabledFiledsView"
                  url="/selectInfiniteCie10" array-info="cie10" :itemsData="cie10_arrayInfo" :firstFetch="false" />
              </VCol>

              <VCol cols="12" md="6">
                <AppSelectRemote clearable label="codComplicacion" v-model="form.codComplicacion_id"
                  :requiredField="true" :rules="[requiredValidator]" :error-messages="errorsBack.codComplicacion_id"
                  @input="errorsBack.codComplicacion_id = ''" :disabled="disabledFiledsView" url="/selectInfiniteCie10"
                  array-info="cie10" :itemsData="cie10_arrayInfo" :firstFetch="false" />
              </VCol>

              <VCol cols="12" md="6">
                <FormatCurrency clearable label="valorPagoModerador" v-model="form.valorPagoModerador"
                  :rules="[positiveValidator, lessThanVrService]" :error-messages="errorsBack.valorPagoModerador"
                  @input="errorsBack.valorPagoModerador = ''" :disabled="disabledFiledsView"
                  @realValue="dataReal($event, 'real_valorPagoModerador')" />
              </VCol>
              <VCol cols="12" md="6">
                <FormatCurrency clearable label="vrServicio" v-model="form.vrServicio" :requiredField="true"
                  :rules="[requiredValidator, positiveValidator]" :error-messages="errorsBack.vrServicio"
                  @input="errorsBack.vrServicio = ''" :disabled="disabledFiledsView"
                  @realValue="dataReal($event, 'real_vrServicio')" />
              </VCol>

              <VCol cols="12" md="6">
                <AppSelectRemote clearable label="conceptoRecaudo" v-model="form.conceptoRecaudo_id"
                  :error-messages="errorsBack.conceptoRecaudo_id" @input="errorsBack.conceptoRecaudo_id = ''"
                  :disabled="disabledFiledsView" url="/selectInfiniteConceptoRecaudo" array-info="conceptoRecaudo"
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
