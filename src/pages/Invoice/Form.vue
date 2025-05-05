<script lang="ts" setup>
import { useToast } from '@/composables/useToast';
import IErrorsBack from "@/interfaces/Axios/IErrorsBack";
import { useAuthenticationStore } from "@/stores/useAuthenticationStore";
import type { VForm } from 'vuetify/components/VForm';

const authenticationStore = useAuthenticationStore();

definePage({
  path: "invoice-Form/:action/:type?/:id?",
  name: "Invoice-Form",
  meta: {
    redirectIfLoggedIn: true,
    requiresAuth: true,
    requiredPermission: "menu.invoice",
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

const form = ref({
  id: null as string | null,
  serviceVendor: null as string | null,
  entity: null as string | null,
  invoice_number: null as string | null,
  value_glosa: null as string | null,
  value_approved: null as string | null,
  invoice_date: null as string | null,
  radication_date: null as string | null,
})

const typeEntities = ref<Array<object>>([])

const clearForm = () => {
  for (const key in form.value) {
    form.value[key] = null
  }
}

const fetchDataForm = async () => {

  form.value.id = route.params.id || null

  const url = form.value.id ? `/invoice/${form.value.id}/edit` : `/invoice/create`

  loading.form = true
  const { response, data } = await useAxios(url).get();

  if (response.status == 200 && data) {
    typeEntities.value = data.typeEntities
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

    const url = form.value.id ? `/invoice/update/${form.value.id}` : `/invoice/store`

    form.value.company_id = authenticationStore.company.id;

    loading.form = true;
    const { data, response } = await useAxios(url).post(form.value);

    if (response.status == 200 && data) {

      // if (data.code == 200) {
      //   router.push({ name: 'Entity-List' })
      // }
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

const company = {
  company_id: authenticationStore.company.id,
}

const entityData = ref({
  nit: null,
});

const changeEntity = async (event: any) => {
  if (!event) return;
  loading.form = true;

  const url = `/entity/getNit/${event.value}`;
  const { response, data } = await useAxios(url).get({
    params: { company_id: authenticationStore.company.id }
  });
  if (response.status === 200 && data) {
    entityData.value.nit = data.nit;
  }
  loading.form = false;
};
</script>


<template>
  <div>
    <VCard :disabled="loading.form" :loading="loading.form">
      <VCardTitle class="d-flex justify-space-between">
        <span>
          Formulario Factura
        </span>
      </VCardTitle>

      <VCardText>
        <VForm ref="formValidation" @submit.prevent="() => { }" :disabled="disabledFiledsView">
          <VRow>
            <VCol cols="12" sm="4">
              <AppSelectRemote label="Prestador" v-model="form.serviceVendor" url="/selectInfiniteServiceVendor"
                arrayInfo="serviceVendors" :requiredField="true" :rules="[requiredValidator]" clearable
                :params="company">
              </AppSelectRemote>
            </VCol>

            <VCol cols="12" sm="4">
              <AppSelectRemote label="Entidad" v-model="form.entity" url="/selectInfiniteEntities" arrayInfo="entities"
                :requiredField="true" @update:model-value="changeEntity($event)" :rules="[requiredValidator]" clearable
                :params="company">
              </AppSelectRemote>
            </VCol>

            <VCol cols="12" sm="4">
              <AppTextField :disabled="true" :rules="[requiredValidator]" v-model="entityData.nit" label="Nit Entidad"
                clearable />
            </VCol>

            <VCol cols="12" sm="4">
              <AppTextField :requiredField="true" :rules="[requiredValidator]" v-model="form.invoice_number"
                label="Número de Factura" :error-messages="errorsBack.invoice_number"
                @input="errorsBack.invoice_number = ''" clearable />
            </VCol>

            <VCol cols="12" sm="4">
              <AppTextField :requiredField="true" :rules="[requiredValidator]" v-model="form.value_glosa"
                label="Valor Glosado" :error-messages="errorsBack.value_glosa" @input="errorsBack.value_glosa = ''"
                clearable />
            </VCol>

            <VCol cols="12" sm="4">
              <AppTextField :requiredField="true" :rules="[requiredValidator]" v-model="form.value_approved"
                label="Valor Aprobado" :error-messages="errorsBack.value_approved"
                @input="errorsBack.value_approved = ''" clearable />
            </VCol>

            <VCol cols="12" sm="4">
              <AppDateTimePicker clearable :requiredField="true" label="Fecha Factura" v-model="form.invoice_date"
                :error-messages="errorsBack.invoice_date" :rules="[requiredValidator]"
                :config="{ dateFormat: 'Y-m-d' }" />
            </VCol>

            <VCol cols="12" sm="4">
              <AppDateTimePicker clearable :requiredField="true" label="Fecha Radicación" v-model="form.radication_date"
                :error-messages="errorsBack.radication_date" :rules="[requiredValidator]"
                :config="{ dateFormat: 'Y-m-d' }" />
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
