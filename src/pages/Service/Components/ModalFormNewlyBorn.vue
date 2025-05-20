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

const titleModal = ref<string>("Servicio: Recien Nacidos")
const isDialogVisible = ref<boolean>(false)
const disabledFiledsView = ref<boolean>(false)
const isLoading = ref<boolean>(false)

const cie10_arrayInfo = ref([])
const condicionyDestinoUsuarioEgreso_arrayInfo = ref([])
const sexos_arrayInfo = ref([])
const tipoIdPisis_arrayInfo = ref([])

const service_id = ref<null | string>(null)

const form = ref({
  id: null as string | null,
  invoice_id: null as string | null,

  fechaNacimiento: null as string | null,
  edadGestacional: null as string | null,
  numConsultasCPrenatal: null as string | null,
  codSexoBiologico_id: null as string | null,
  peso: null as string | null,
  codDiagnosticoPrincipal_id: null as string | null,
  condicionDestino: null as string | null,
  condicionDestinoUsuarioEgreso_id: null as string | null,
  codDiagnosticoCausaMuerte_id: null as string | null,
  fechaEgreso: null as string | null,
  tipoDocumentoIdentificacion_id: null as string | null,
  numDocumentoIdentificacion: null as string | null,
  numFEVPagoModerador: null as string | null,
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
    const url = service_id.value ? `/service/newlyBorn/${service_id.value}/edit` : `/service/newlyBorn/create`;
    const { data, response } = await useAxios(url).get();

    if (response.status === 200 && data) {

      cie10_arrayInfo.value = data.cie10_arrayInfo
      condicionyDestinoUsuarioEgreso_arrayInfo.value = data.condicionyDestinoUsuarioEgreso_arrayInfo
      sexos_arrayInfo.value = data.sexos_arrayInfo
      tipoIdPisis_arrayInfo.value = data.tipoIdPisis_arrayInfo

      if (data.form) {
        form.value = cloneObject(data.form);
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
    const url = form.value.id ? `/service/newlyBorn/update/${form.value.id}` : `/service/newlyBorn/store`;

    const payload = {
      ...form.value,
      service_id: service_id.value,
      company_id: authenticationStore.company.id,
    };

    const { data, response } = await useAxios(url).post(payload);

    if (response.status === 200 && data.code == 200) {
      if (!form.value.id) {
        emit("created")
      }
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
                <AppTextField clearable label="fechaNacimiento" v-model="form.fechaNacimiento" :requiredField="true"
                  :rules="[requiredValidator]" :error-messages="errorsBack.fechaNacimiento"
                  @input="errorsBack.fechaNacimiento = ''" :disabled="disabledFiledsView" type="date" />
              </VCol>

              <VCol cols="12" md="6">
                <AppTextField clearable label="edadGestacional" v-model="form.edadGestacional" :requiredField="true"
                  :rules="[requiredValidator]" :error-messages="errorsBack.edadGestacional"
                  @input="errorsBack.edadGestacional = ''" :disabled="disabledFiledsView" />
              </VCol>

              <VCol cols="12" md="6">
                <AppTextField clearable label="numConsultasCPrenatal" v-model="form.numConsultasCPrenatal"
                  :requiredField="true" :rules="[requiredValidator]" :error-messages="errorsBack.numConsultasCPrenatal"
                  @input="errorsBack.numConsultasCPrenatal = ''" :disabled="disabledFiledsView" />
              </VCol>

              <VCol cols="12" md="6">
                <AppSelectRemote clearable label="codSexoBiologico" v-model="form.codSexoBiologico_id"
                  :requiredField="true" :rules="[requiredValidator]" :error-messages="errorsBack.codSexoBiologico_id"
                  @input="errorsBack.codSexoBiologico_id = ''" :disabled="disabledFiledsView" url="/selectInfiniteSexo"
                  array-info="sexos" :itemsData="sexos_arrayInfo" :firstFetch="false" />
              </VCol>

              <VCol cols="12" md="6">
                <AppTextField clearable label="peso" v-model="form.peso" :requiredField="true"
                  :rules="[requiredValidator]" :error-messages="errorsBack.peso" @input="errorsBack.peso = ''"
                  :disabled="disabledFiledsView" />
              </VCol>

              <VCol cols="12" md="6">
                <AppSelectRemote clearable label="codDiagnosticoPrincipal" v-model="form.codDiagnosticoPrincipal_id"
                  :requiredField="true" :rules="[requiredValidator]"
                  :error-messages="errorsBack.codDiagnosticoPrincipal_id"
                  @input="errorsBack.codDiagnosticoPrincipal_id = ''" :disabled="disabledFiledsView"
                  url="/selectInfiniteCie10" array-info="cie10" :itemsData="cie10_arrayInfo" :firstFetch="false" />
              </VCol>

              <VCol cols="12" md="6">
                <AppTextField clearable label="condicionDestino" v-model="form.condicionDestino" :requiredField="true"
                  :rules="[requiredValidator]" :error-messages="errorsBack.condicionDestino"
                  @input="errorsBack.condicionDestino = ''" :disabled="disabledFiledsView" />
              </VCol>

              <VCol cols="12" md="6">
                <AppSelectRemote clearable label="condicionDestinoUsuarioEgreso"
                  v-model="form.condicionDestinoUsuarioEgreso_id" :requiredField="true" :rules="[requiredValidator]"
                  :error-messages="errorsBack.condicionDestinoUsuarioEgreso_id"
                  @input="errorsBack.condicionDestinoUsuarioEgreso_id = ''" :disabled="disabledFiledsView"
                  url="/selectInfiniteCondicionyDestinoUsuarioEgreso" array-info="condicionyDestinoUsuarioEgreso"
                  :itemsData="condicionyDestinoUsuarioEgreso_arrayInfo" :firstFetch="false" />
              </VCol>

              <VCol cols="12" md="6">
                <AppSelectRemote clearable label="codDiagnosticoCausaMuerte" v-model="form.codDiagnosticoCausaMuerte_id"
                  :requiredField="true" :rules="[requiredValidator]"
                  :error-messages="errorsBack.codDiagnosticoCausaMuerte_id"
                  @input="errorsBack.codDiagnosticoCausaMuerte_id = ''" :disabled="disabledFiledsView"
                  url="/selectInfiniteCie10" array-info="cie10" :itemsData="cie10_arrayInfo" :firstFetch="false" />
              </VCol>

              <VCol cols="12" md="6">
                <AppTextField clearable label="fechaEgreso" v-model="form.fechaEgreso" :requiredField="true"
                  :rules="[requiredValidator]" :error-messages="errorsBack.fechaEgreso"
                  @input="errorsBack.fechaEgreso = ''" :disabled="disabledFiledsView" type="date" />
              </VCol>


              <VCol cols="12" md="6">
                <AppSelectRemote clearable label="Tipo documento del recien nacido"
                  v-model="form.tipoDocumentoIdentificacion_id" :requiredField="true" :rules="[requiredValidator]"
                  :error-messages="errorsBack.tipoDocumentoIdentificacion_id"
                  :params="{ codigo_in: ['CN', 'RC', 'MS'] }" @input="errorsBack.tipoDocumentoIdentificacion_id = ''"
                  :disabled="disabledFiledsView" url="/selectInfiniteTipoIdPisis" array-info="tipoIdPisis"
                  :itemsData="tipoIdPisis_arrayInfo" :firstFetch="false" />
              </VCol>

              <VCol cols="12" md="6">
                <AppTextField clearable label="Número documento del recien nacido"
                  v-model="form.numDocumentoIdentificacion" :requiredField="true" :rules="[requiredValidator]"
                  :error-messages="errorsBack.numDocumentoIdentificacion"
                  @input="errorsBack.numDocumentoIdentificacion = ''" :disabled="disabledFiledsView" />
              </VCol>

              <VCol cols="12" md="6">
                <AppTextField clearable label="numFEVPagoModerador" v-model="form.numFEVPagoModerador"
                  :requiredField="true" :rules="[requiredValidator]" :error-messages="errorsBack.numFEVPagoModerador"
                  @input="errorsBack.numFEVPagoModerador = ''" :disabled="disabledFiledsView" />
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
