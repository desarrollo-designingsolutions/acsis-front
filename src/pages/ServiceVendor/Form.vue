<script lang="ts" setup>
import { useToast } from '@/composables/useToast';
import IErrorsBack from "@/interfaces/Axios/IErrorsBack";
import { router } from '@/plugins/1.router';
import { useAuthenticationStore } from "@/stores/useAuthenticationStore";
import type { VForm } from 'vuetify/components/VForm';

definePage({
  path: "serviceVendor-form/:action/:id?",
  name: "ServiceVendor-Form",
  meta: {
    redirectIfLoggedIn: true,
    requiresAuth: true,
    requiredPermission: "serviceVendor.list",
  },
});

const authenticationStore = useAuthenticationStore();

const ipsNoReps_arrayInfo = ref([])

const { toast } = useToast()
const errorsBack = ref<IErrorsBack>({});
const disabledFiledsView = ref<boolean>(false);
const route = useRoute()
const formValidation = ref<VForm>()
const loading = reactive({
  form: false,
})

const form = ref({
  id: null as string | null,
  name: null as string | null,
  nit: null as string | null,
  phone: null as string | null,
  address: null as string | null,
  email: null as string | null,
  ips_no_rep_id: null as string | null,
})

const clearForm = () => {
  for (const key in form.value) {
    form.value[key] = null
  }
}

const fetchDataForm = async () => {

  form.value.id = route.params.id || null

  form.value.company_id = authenticationStore.company.id

  const url = form.value.id ? `/serviceVendor/${form.value.id}/edit` : `/serviceVendor/create`

  loading.form = true
  const { response, data } = await useAxios(url).get();

  if (response.status == 200 && data) {
    type_vendors.value = data.type_vendors
    ipsNoReps_arrayInfo.value = data.ipsNoReps_arrayInfo

    //formulario 
    if (data.form) {
      form.value = cloneObject(data.form)
      tabs.value[1].show = true;
    }
  }
  loading.form = false
}

const submitForm = async () => {
  const validation = await formValidation.value?.validate()
  if (validation?.valid) {

    const url = form.value.id ? `/serviceVendor/update/${form.value.id}` : `/serviceVendor/store`

    loading.form = true;
    const { data, response } = await useAxios(url).post(form.value);

    if (response.status == 200 && data) {

      if (data.code == 200) {
        form.value.id = data.form.id
        tabs.value[1].show = true;
        router.replace({ name: "ServiceVendor-Form", params: { action: "edit", id: data.form.id } });
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

//ModalQuestion
const refModalQuestion = ref()

const openModalQuestion = (response: IImageSelected) => {
  refModalQuestion.value.componentData.isDialogVisible = true
  refModalQuestion.value.componentData.dialogMaxWidth = '20rem'
  refModalQuestion.value.componentData.showBtnCancel = false
  refModalQuestion.value.componentData.btnSuccessText = 'Ok'
  refModalQuestion.value.componentData.icon = response.icon
  refModalQuestion.value.componentData.title = response.title
  refModalQuestion.value.componentData.subTitle = response.text
}
const type_vendors = ref([])

//TABS
const currentTab = ref(0)

const tabs = ref([
  {
    title: "Información general",
    show: true,
    errorsValidations: false,
  },
  {
    title: "Documentos",
    show: false,
    errorsValidations: false,
  },
])

const nitRules = [
  value => (!value || /^[0-9]{9}-[0-9]{1}$/.test(value)) || 'El NIT debe tener el formato 000000000-0',
  value => requiredValidator(value),
];
const phoneRules = [
  value => requiredValidator(value),
  value => integerValidator(value),
  value => (!value || value.length <= 10) || "El número no debe tener mas de 10 caracteres",
  value => positiveNumberValidator(value),
];
</script>


<template>
  <div>
    <VCard :disabled="loading.form" :loading="loading.form">
      <VCardTitle class="d-flex justify-space-between">
        <span>
          Formulario prestador
        </span>
      </VCardTitle>


      <VCardText>
        <VTabs v-model="currentTab">
          <VTab class="text-none" v-for="(item, index) in tabs" :key="index" v-show="item.show">
            <VIcon start :icon="!item.errorsValidations ? '' : 'tabler-alert-circle-filled'"
              :color="!item.errorsValidations ? '' : 'error'" />
            {{ item.title }}
          </VTab>
        </VTabs>
      </VCardText>

      <VCardText>
        <VWindow v-model="currentTab" class="my-5">
          <VWindowItem>
            <VForm ref="formValidation" @submit.prevent="() => { }" :disabled="disabledFiledsView">
              <VRow>
                <VCol sm="4">
                  <AppTextField :requiredField="true" :rules="[requiredValidator]" v-model="form.name"
                    label="Razón Social" :error-messages="errorsBack.name" @input="errorsBack.name = ''" clearable />
                </VCol>
                <VCol sm="4">
                  <AppTextField :requiredField="true" :rules="nitRules" v-model="form.nit" label="Nit"
                    :error-messages="errorsBack.nit" @input="errorsBack.nit = ''" clearable />
                </VCol>
                <VCol sm="4">
                  <AppSelect :requiredField="true" :items="type_vendors" :rules="[requiredValidator]"
                    v-model="form.type_vendor_id" label="Tipo" :error-messages="errorsBack.type_vendor_id"
                    @input="errorsBack.type_vendor_id = ''" clearable />
                </VCol>
                <VCol sm="4">
                  <AppTextField :requiredField="true" :rules="[requiredValidator]" clearable v-model="form.address"
                    label="Dirección" :error-messages="errorsBack.address" />
                </VCol>
                <VCol sm="4">
                  <AppTextField :requiredField="true" :rules="phoneRules" v-model="form.phone" label="Teléfono"
                    :error-messages="errorsBack.phone" @input="errorsBack.phone = ''" />
                </VCol>
                <VCol sm="4">
                  <AppTextField :requiredField="true" clearable :rules="[requiredValidator, emailValidator]"
                    v-model="form.email" label="Correo de contacto" :error-messages="errorsBack.email"
                    @input="errorsBack.email = ''" />
                </VCol>

                <VCol sm="4">
                  <AppSelectRemote clearable label="Código habilitación" v-model="form.ips_no_rep_id"
                    :requiredField="true" :rules="[requiredValidator]" :error-messages="errorsBack.ips_no_rep_id"
                    @input="errorsBack.ips_no_rep_id = ''" :disabled="disabledFiledsView" url="/selectInfiniteIpsNoReps"
                    array-info="ipsNoReps" :itemsData="ipsNoReps_arrayInfo" :firstFetch="false" />
                </VCol>

              </VRow>
            </VForm>
          </VWindowItem>

          <VWindowItem>
            <Files v-if="form.id" model="ServiceVendor" :id="form.id" :disabled="disabledFiledsView"></Files>
          </VWindowItem>

        </VWindow>

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
