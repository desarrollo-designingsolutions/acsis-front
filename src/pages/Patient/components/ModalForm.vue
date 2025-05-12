<script lang="ts" setup>
import { useToast } from '@/composables/useToast';
import IErrorsBack from "@/interfaces/Axios/IErrorsBack";
import { useAuthenticationStore } from "@/stores/useAuthenticationStore";
import type { VForm } from 'vuetify/components/VForm';

const authenticationStore = useAuthenticationStore();

const emit = defineEmits(["closeModal"]);

const { toast } = useToast()
const errorsBack = ref<IErrorsBack>({});
const disabledFiledsView = ref<boolean>(false);
const formValidation = ref<VForm>()
const loading = reactive({
  form: false,
})

const tipoIdPisiss_arrayInfo = ref([])
const ripsTipoUsuarioVersion2s_arrayInfo = ref([])
const sexos_arrayInfo = ref([])
const municipios_arrayInfo = ref([])
const paises_arrayInfo = ref([])
const zonaVersion2s_arrayInfo = ref([])

const form = ref({
  id: null as string | null,
  tipo_id_pisi_id: null as string | null,
  document: null as string | null,
  rips_tipo_usuario_version2_id: null as string | null,
  birth_date: null as string | null,
  sexo_id: null as string | null,
  pais_residency_id: null as string | null,
  municipio_residency_id: null as string | null,
  zona_version2_id: null as string | null,
  incapacity: null as string | null,
  pais_origin_id: null as string | null,
  first_name: null as string | null,
  second_name: null as string | null,
  first_surname: null as string | null,
  second_surname: null as string | null,
})

const handleClearForm = () => {
  for (const key in form.value) {
    form.value[key] = null
  }
}

const fetchDataForm = async () => {

  const url = form.value.id ? `/patient/${form.value.id}/edit` : `/patient/create`

  loading.form = true
  const { response, data } = await useAxios(url).get();

  if (response.status == 200 && data) {
    tipoIdPisiss_arrayInfo.value = data.tipoIdPisiss_arrayInfo
    ripsTipoUsuarioVersion2s_arrayInfo.value = data.ripsTipoUsuarioVersion2s_arrayInfo
    sexos_arrayInfo.value = data.sexos_arrayInfo
    municipios_arrayInfo.value = data.municipios_arrayInfo
    paises_arrayInfo.value = data.paises_arrayInfo
    zonaVersion2s_arrayInfo.value = data.zonaVersion2s_arrayInfo

    //formulario 
    if (data.form) {
      form.value = cloneObject(data.form)
    }
  }
  loading.form = false
}

const submitForm = async () => {
  const validation = await formValidation.value?.validate()
  if (validation?.valid) {

    const url = form.value.id ? `/patient/update/${form.value.id}` : `/patient/store`

    form.value.company_id = authenticationStore.company.id;

    loading.form = true;
    const { data, response } = await useAxios(url).post(form.value);

    if (response.status == 200 && data) {

      if (data.code == 200) {
        emit("closeModal", data.data)
        handleDialogVisible()
      }
    }
    if (data.code === 422) errorsBack.value = data.errors ?? {};

    loading.form = false;
  }
  else {
    toast('Faltan Campos Por Diligenciar', '', 'danger')
  }
}

const isDialogVisible = ref<boolean>(false)
const handleDialogVisible = () => {
  isDialogVisible.value = !isDialogVisible.value;
};

// Computed que verifica si al menos uno de los valores es true
const isLoading = computed(() => {
  return Object.values(loading).some(value => value);
});

const openModal = async (id: any = null, disabled: boolean = false) => {
  handleClearForm();
  handleDialogVisible();

  disabledFiledsView.value = disabled;
  form.value.id = id;

  await fetchDataForm();
};

defineExpose({
  openModal,
});
</script>


<template>
  <VDialog v-model="isDialogVisible" :overlay="false" max-width="60rem" transition="dialog-transition" persistent>
    <DialogCloseBtn @click="handleDialogVisible" />
    <div>
      <VCard :disabled="loading.form" :loading="loading.form">

        <div>
          <VToolbar color="primary">
            <VToolbarTitle>Formulario Pacientes</VToolbarTitle>
          </VToolbar>
        </div>

        <VCardText>
          <VForm ref="formValidation" @submit.prevent="() => { }" :disabled="disabledFiledsView">
            <VRow>
              <VCol cols="12" sm="4">
                <AppSelectRemote :disabled="disabledFiledsView" label="Tipo de Documento" v-model="form.tipo_id_pisi_id"
                  url="/selectInfiniteTipoDocumento" arrayInfo="tipoIdPisiss" :requiredField="true"
                  :rules="[requiredValidator]" clearable :itemsData="tipoIdPisiss_arrayInfo" :firstFetch="false">
                </AppSelectRemote>
              </VCol>

              <VCol cols="12" sm="4">
                <AppTextField :requiredField="true" :rules="[requiredValidator]" clearable v-model="form.document"
                  label="Documento" :error-messages="errorsBack.document" />
              </VCol>

              <VCol cols="12" sm="4">
                <AppSelectRemote :disabled="disabledFiledsView" label="Tipo de Usuario"
                  v-model="form.rips_tipo_usuario_version2_id" url="/selectInfiniteTipoUsuario"
                  arrayInfo="ripsTipoUsuarioVersion2s" :requiredField="true" :rules="[requiredValidator]" clearable
                  :itemsData="ripsTipoUsuarioVersion2s_arrayInfo" :firstFetch="false">
                </AppSelectRemote>
              </VCol>

              <VCol cols="12" sm="4">
                <AppTextField clearable type="date" :requiredField="true" :error-messages="errorsBack.birth_date"
                  :rules="[requiredValidator]" v-model="form.birth_date" label="Fecha de Nacimiento"
                  @input="errorsBack.birth_date = ''" />
              </VCol>

              <VCol cols="12" sm="4">
                <AppSelectRemote :disabled="disabledFiledsView" label="Sexo" v-model="form.sexo_id"
                  url="/selectInfiniteSexo" arrayInfo="sexos" :requiredField="true" :rules="[requiredValidator]"
                  clearable :itemsData="sexos_arrayInfo" :firstFetch="false">
                </AppSelectRemote>
              </VCol>

              <VCol cols="12" sm="4">
                <AppSelectRemote :disabled="disabledFiledsView" label="Pais de Residencia"
                  v-model="form.pais_residency_id" url="/selectInfinitePais" arrayInfo="paises" :requiredField="true"
                  :rules="[requiredValidator]" clearable :itemsData="paises_arrayInfo" :firstFetch="false">
                </AppSelectRemote>
              </VCol>

              <VCol cols="12" sm="4">
                <AppSelectRemote :disabled="disabledFiledsView" label="Municipio de Residencia"
                  v-model="form.municipio_residency_id" url="/selectInfiniteMunicipio" arrayInfo="municipios"
                  :requiredField="true" :rules="[requiredValidator]" clearable :itemsData="municipios_arrayInfo"
                  :firstFetch="false">
                </AppSelectRemote>
              </VCol>

              <VCol cols="12" sm="4">
                <AppSelectRemote :disabled="disabledFiledsView" label="Zona Territorial de Residencia"
                  v-model="form.zona_version2_id" url="/selectInfiniteZonaVersion2" arrayInfo="zonaVersion2s"
                  :requiredField="true" :rules="[requiredValidator]" clearable :itemsData="zonaVersion2s_arrayInfo"
                  :firstFetch="false">
                </AppSelectRemote>
              </VCol>

              <VCol cols="12" sm="4">
                <AppSelect v-model="form.incapacity" label="Incapacidad" :items="['Si', 'No']" />
              </VCol>

              <VCol cols="12" sm="4">
                <AppSelectRemote :disabled="disabledFiledsView" label="Pais de Origen" v-model="form.pais_origin_id"
                  url="/selectInfinitePais" arrayInfo="paises" :requiredField="true" :rules="[requiredValidator]"
                  clearable :itemsData="paises_arrayInfo" :firstFetch="false">
                </AppSelectRemote>
              </VCol>

              <VCol cols="12" sm="4">
                <AppTextField :requiredField="true" :rules="[requiredValidator]" clearable v-model="form.first_name"
                  label="Primer Nombre" :error-messages="errorsBack.first_name" />
              </VCol>

              <VCol cols="12" sm="4">
                <AppTextField clearable v-model="form.second_name" label="Segundo Nombre"
                  :error-messages="errorsBack.second_name" />
              </VCol>

              <VCol cols="12" sm="4">
                <AppTextField :requiredField="true" :rules="[requiredValidator]" clearable v-model="form.first_surname"
                  label="Primer Apellido" :error-messages="errorsBack.first_surname" />
              </VCol>

              <VCol cols="12" sm="4">
                <AppTextField clearable v-model="form.second_surname" label="Segundo Apellido"
                  :error-messages="errorsBack.second_surname" />
              </VCol>
            </VRow>
          </VForm>
        </VCardText>

        <VCardText class="d-flex justify-end gap-3 flex-wrap mt-5">
          <VBtn :disabled="isLoading" :loading="isLoading" @click="handleDialogVisible()" color="secondary">
            Cancelar
          </VBtn>
          <VBtn v-if="!disabledFiledsView" :disabled="isLoading" :loading="isLoading" @click="submitForm()"
            color="primary">
            Guardar
          </VBtn>
        </VCardText>
      </VCard>

      <ModalQuestion ref="refModalQuestion" />
    </div>
  </VDialog>
</template>
