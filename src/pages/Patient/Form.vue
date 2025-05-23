<script lang="ts" setup>
import { useToast } from '@/composables/useToast';
import IErrorsBack from "@/interfaces/Axios/IErrorsBack";
import { router } from '@/plugins/1.router';
import { useAuthenticationStore } from "@/stores/useAuthenticationStore";
import type { VForm } from 'vuetify/components/VForm';

const authenticationStore = useAuthenticationStore();

definePage({
  path: "patient-Form/:action/:id?",
  name: "Patient-Form",
  meta: {
    redirectIfLoggedIn: true,
    requiresAuth: true,
    requiredPermission: "menu.patient",
  },
});

const { toast } = useToast()
const errorsBack = ref<IErrorsBack>({});
const disabledFiledsView = ref<boolean>(false);
const route = useRoute()
const formValidation = ref<VForm>()
const loading = reactive({
  form: false,
})

const tipoIdPisis_arrayInfo = ref([])
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

const clearForm = () => {
  for (const key in form.value) {
    form.value[key] = null
  }
}

const fetchDataForm = async () => {

  form.value.id = route.params.id || null

  const url = form.value.id ? `/patient/${form.value.id}/edit` : `/patient/create`

  loading.form = true
  const { response, data } = await useAxios(url).get();

  if (response.status == 200 && data) {
    tipoIdPisis_arrayInfo.value = data.tipoIdPisis_arrayInfo
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
        router.push({ name: 'Patient-List' })
      }
    }
    if (data.code === 422) errorsBack.value = data.errors ?? {};

    loading.form = false;
  }
  else {
    toast('Faltan Campos Por Diligenciar', '', 'danger')
  }
}

if (route.params.action == 'view') disabledFiledsView.value = true

onMounted(async () => {
  clearForm()

  await fetchDataForm()
})

// Computed que verifica si al menos uno de los valores es true
const isLoading = computed(() => {
  return Object.values(loading).some(value => value);
});

const company = authenticationStore.company.id;

// Validations
const documentRules = [
  value => lengthBetweenValidator(value, 4, 20),
  value => requiredValidator(value),
];

</script>


<template>
  <div>
    <VCard :disabled="loading.form" :loading="loading.form">
      <VCardTitle class="d-flex justify-space-between">
        <span>
          Formulario Pacientes
        </span>
      </VCardTitle>

      <VCardText>
        <VForm ref="formValidation" @submit.prevent="() => { }" :disabled="disabledFiledsView">
          <VRow>
            <VCol cols="12" sm="4">
              <AppSelectRemote label="Tipo de Documento" v-model="form.tipo_id_pisi_id" url="/selectInfiniteTipoIdPisis"
                arrayInfo="tipoIdPisis" :requiredField="true" :rules="[requiredValidator]" clearable :params="company"
                :itemsData="tipoIdPisis_arrayInfo" :firstFetch="false">
              </AppSelectRemote>
            </VCol>

            <VCol cols="12" sm="4">
              <AppTextField :requiredField="true" :rules="documentRules" clearable v-model="form.document"
                label="Documento" :error-messages="errorsBack.document" counter maxlength="20" minlength="4" />
            </VCol>

            <VCol cols="12" sm="4">
              <AppSelectRemote label="Tipo de Usuario" v-model="form.rips_tipo_usuario_version2_id"
                url="/selectInfiniteTipoUsuario" arrayInfo="ripsTipoUsuarioVersion2s" :requiredField="true"
                :rules="[requiredValidator]" clearable :params="company" :itemsData="ripsTipoUsuarioVersion2s_arrayInfo"
                :firstFetch="false">
              </AppSelectRemote>
            </VCol>

            <VCol cols="12" sm="4">
              <AppDateTimePicker clearable :requiredField="true" label="Fecha de Nacimiento" v-model="form.birth_date"
                :error-messages="errorsBack.birth_date" :rules="[requiredValidator]"
                :config="{ dateFormat: 'Y-m-d' }" />
            </VCol>

            <VCol cols="12" sm="4">
              <AppSelectRemote label="Sexo" v-model="form.sexo_id" url="/selectInfiniteSexo" arrayInfo="sexos"
                :requiredField="true" :rules="[requiredValidator]" clearable :params="company"
                :itemsData="sexos_arrayInfo" :firstFetch="false">
              </AppSelectRemote>
            </VCol>

            <VCol cols="12" sm="4">
              <AppSelectRemote label="Pais de Residencia" v-model="form.pais_residency_id" url="/selectInfinitePais"
                arrayInfo="paises" :requiredField="true" :rules="[requiredValidator]" clearable :params="company"
                :itemsData="paises_arrayInfo" :firstFetch="false">
              </AppSelectRemote>
            </VCol>

            <VCol cols="12" sm="4">
              <AppSelectRemote label="Municipio de Residencia" v-model="form.municipio_residency_id"
                url="/selectInfiniteMunicipio" arrayInfo="municipios" :requiredField="true" :rules="[requiredValidator]"
                clearable :params="company" :itemsData="municipios_arrayInfo" :firstFetch="false">
              </AppSelectRemote>
            </VCol>

            <VCol cols="12" sm="4">
              <AppSelectRemote label="Zona Territorial de Residencia" v-model="form.zona_version2_id"
                url="/selectInfiniteZonaVersion2" arrayInfo="zonaVersion2s" :requiredField="true"
                :rules="[requiredValidator]" clearable :params="company" :itemsData="zonaVersion2s_arrayInfo"
                :firstFetch="false">
              </AppSelectRemote>
            </VCol>

            <VCol cols="12" sm="4">
              <AppSelect v-model="form.incapacity" label="Incapacidad" :items="['SÍ', 'NO']" />
            </VCol>

            <VCol cols="12" sm="4">
              <AppSelectRemote label="Pais de Origen" v-model="form.pais_origin_id" url="/selectInfinitePais"
                arrayInfo="paises" :requiredField="true" :rules="[requiredValidator]" clearable :params="company"
                :itemsData="paises_arrayInfo" :firstFetch="false">
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
        <BtnBack :disabled="isLoading" :loading="isLoading" />
        <VBtn v-if="!disabledFiledsView" :disabled="isLoading" :loading="isLoading" @click="submitForm()"
          color="primary">
          Guardar
        </VBtn>
      </VCardText>
    </VCard>

    <ModalQuestion ref="refModalQuestion" />
  </div>
</template>
