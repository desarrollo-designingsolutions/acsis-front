<script setup lang="ts">
import { useToast } from '@/composables/useToast';
import IErrorsBack from '@/interfaces/Axios/IErrorsBack';
import { router } from '@/plugins/1.router';
import { useAuthenticationStore } from "@/stores/useAuthenticationStore";
import { reactive, ref } from 'vue';
import type { VForm } from 'vuetify/components/VForm';

const { toast } = useToast()

definePage({
  path: 'invoice-Furips2/:invoice_id/:id?',
  name: 'Invoice-Furips2',
  meta: {
    redirectIfLoggedIn: true,
    requiresAuth: true,
    requiredPermission: 'invoice.furips2',
  },
});

interface IInvoiceData {
  id: string | null;
  furips1_consecutiveClaimNumber: string | null;
}
interface ISelect {
  title: string;
  value: string;
}
interface CodTecnologiaSaludablesSelect {
  label: string;
  url: string;
  arrayInfo: string;
  itemsData: any[];
}
interface IForm {
  id: null | string;
  invoice_id: null | string;
  consecutiveNumberClaim: string | null;
  serviceType: string | null;
  serviceCode_type: string | null;
  serviceCode_id: string | null;
  serviceDescription: string | null;
  serviceDate: string | null;
  serviceValue: string | null;
  totalFactoryValue: string | null;
  totalClaimedValue: string | null;
  victimResidenceAddress: string | null; // Existing field
}

const form = ref<IForm>({
  id: null,
  invoice_id: null,
  consecutiveNumberClaim: null,
  serviceType: null,
  serviceCode_type: null,
  serviceCode_id: null,
  serviceDescription: null,
  serviceDate: null,
  serviceValue: null,
  totalFactoryValue: null,
  totalClaimedValue: null,
  victimResidenceAddress: null,
});

const clearForm = () => {
  for (const key in form.value) {
    form.value[key] = null
  }
}

const errorsBack = ref<IErrorsBack>({});
const loading = reactive({ form: false });
const disabledFiledsView = ref(false);
const route = useRoute()
const authenticationStore = useAuthenticationStore();
const formValidation = ref<VForm>()

// Computed que verifica si al menos uno de los valores es true
const isLoading = computed(() => {
  return Object.values(loading).some(value => value);
});

const submitForm = async () => {
  const validation = await formValidation.value?.validate()

  console.log("validation", validation);

  if (validation?.valid) {
    const url = form.value.id ? `/furips2/update/${form.value.id}` : `/furips2/store`

    loading.form = true;
    const { data, response } = await useAxios(url).post(form.value);
    loading.form = false;

    if (response.status == 200 && data) {
      if (data.code == 200) {
        form.value.id = data.furips2.id
        router.replace({ name: "Invoice-Furips2", params: { invoice_id: form.value.invoice_id, id: form.value.id } });
      }
    }
    if (data.code === 422) errorsBack.value = data.errors ?? {};

  } else {
    toast('Faltan Campos Por Diligenciar', '', 'danger')
  }
}

const invoice = ref<IInvoiceData>({
  id: null,
  furips1_consecutiveClaimNumber: null,
});

const serviceTypeEnum_arrayInfo = ref<ISelect[]>([])
const codTecnologiaSaludables = ref<ISelect[]>([])
const decreto780de2026_arrayInfo = ref<ISelect[]>([])

const fetchDataForm = async () => {

  form.value.invoice_id = route.params.invoice_id || null
  form.value.id = route.params.id || null

  const url = form.value.id ? `/furips2/${form.value.id}/edit` : `/furips2/create/${form.value.invoice_id}`

  loading.form = true
  const { response, data } = await useAxios(url).get();
  loading.form = false


  if (response.status == 200 && data) {

    serviceTypeEnum_arrayInfo.value = data.serviceTypeEnum_arrayInfo
    codTecnologiaSaludables.value = data.codTecnologiaSaludables
    decreto780de2026_arrayInfo.value = data.decreto780de2026_arrayInfo

    form.value.serviceCode_type = codTecnologiaSaludables.value.at(0)?.value ?? null;


    invoice.value = data.invoice

    form.value.consecutiveNumberClaim = invoice.value.furips1_consecutiveClaimNumber

    //formulario 
    if (data.form) {
      form.value = cloneObject(data.form)


      if (form.value.serviceType == '1') {
        showServiceCodeType1.value = true
      }
      if (form.value.serviceType == '2') {
        showServiceCodeType2.value = true
      }
    }
  }
}

onMounted(async () => {
  clearForm()
  await fetchDataForm()
})


//Validations
const serviceDescription_validation = computed(() => {
  const rules = [
    value => requiredValidator(value),
  ]

  if (invoice.value && ["3", "4", "5", "6", "7", "8"].includes(form.value.serviceType ?? "")) {
    return {
      rules: [],
      requiredField: false
    }
  }
  return {
    rules: rules,
    requiredField: true
  }
})


const codTecnologiaSaludables_select = computed<CodTecnologiaSaludablesSelect>(() => {

  if (form.value.serviceCode_type) {
    return codTecnologiaSaludables.value.find(item => item.value === form.value.serviceCode_type) || {
      label: "",
      url: "",
      arrayInfo: "",
      itemsData: [],
    };
  }

  return {
    label: "",
    url: "",
    arrayInfo: "",
    itemsData: [],
  };
});

const clearserviceCode_type = () => {
  form.value.serviceCode_id = null
}


const showServiceCodeType1 = ref<boolean>(false)
const showServiceCodeType2 = ref<boolean>(false)

const changeServiceType = (event: any) => {

  // showServiceCodeType1.value = false
  // showServiceCodeType2.value = false

  form.value.serviceCode_type = null
  form.value.serviceCode_id = null

  if (event == '1') {
    showServiceCodeType1.value = true
  } else {

    showServiceCodeType1.value = false
  }
  if (event == '2') {
    showServiceCodeType2.value = true
    form.value.serviceCode_type = 'App\\Models\\Decreto780de2026'
  } else {
    showServiceCodeType2.value = false
  }
}

const codTecnologiaSaludables2222 = computed(() => {
  if (form.value.serviceType == "1") {
    // Omite el último elemento del array
    return codTecnologiaSaludables.value.slice(0, -1);
  } else if (form.value.serviceType == "2") {
    // Omite los dos primeros elementos del array
    return codTecnologiaSaludables.value.slice(2);
  } else {
    // No omite ningún elemento
    return codTecnologiaSaludables.value;
  }
});
</script>

<template>
  <div>
    <VCard :disabled="loading.form" :loading="loading.form">
      <VCardTitle class="d-flex justify-space-between">
        <span>Información del Furips2</span>
      </VCardTitle>

      <VCardText>
        <VForm ref="formValidation" @submit.prevent="() => { }" :disabled="disabledFiledsView">
          <VRow>
            <VCol cols="12" sm="4">
              <AppTextField :requiredField="true" :rules="[requiredValidator]"
                label="Número consecutivo de la reclamación" v-model="form.consecutiveNumberClaim" clearable
                :maxlength="20" counter :errorMessages="errorsBack.consecutiveNumberClaim"
                @input="errorsBack.consecutiveNumberClaim = ''" disabled />
            </VCol>
            <VCol cols="12" sm="4">
              <AppSelect :requiredField="true" :rules="[requiredValidator]" label="Tipo de servicio"
                v-model="form.serviceType" clearable :errorMessages="errorsBack.serviceType"
                @input="errorsBack.serviceType = ''" :items="serviceTypeEnum_arrayInfo"
                @update:modelValue="changeServiceType" />
            </VCol>
            <VCol cols="12" sm="4">
              <!-- <div v-if="showServiceCodeType1"> -->
              <VRadioGroup v-model="form.serviceCode_type" inline>
                <VRadio v-for="(item, index) in codTecnologiaSaludables2222" :key="index" :label="item.label"
                  :value="item.value" @click="clearserviceCode_type" />
              </VRadioGroup>

              <AppSelectRemote clearable label="Código del servicio" v-model="form.serviceCode_id" :requiredField="true"
                :rules="[requiredValidator]" :error-messages="errorsBack.serviceCode_id"
                @input="errorsBack.serviceCode_id = ''" :disabled="disabledFiledsView"
                :url="codTecnologiaSaludables_select.url" :array-info="codTecnologiaSaludables_select.arrayInfo"
                :itemsData="codTecnologiaSaludables_select.itemsData" :firstFetch="false" />
              <!-- </div> -->
              <!-- <div v-if="showServiceCodeType2">
                <AppSelectRemote clearable label="Código del servicio" v-model="form.serviceCode_id"
                  :requiredField="true" :rules="[requiredValidator]" :error-messages="errorsBack.serviceCode_id"
                  @input="errorsBack.serviceCode_id = ''" :disabled="disabledFiledsView"
                  url="/selectInfiniteDecreto780de2026" array-info="decreto780de2026"
                  :itemsData="decreto780de2026_arrayInfo" :firstFetch="false" />
              </div> -->

            </VCol>
            <VCol cols="12" sm="4">
              <AppTextField :requiredField="serviceDescription_validation.requiredField"
                :rules="serviceDescription_validation.rules" label="Descripción del servicio o elemento reclamado"
                v-model="form.serviceDescription" clearable :maxlength="100" counter
                :errorMessages="errorsBack.serviceDescription" @input="errorsBack.serviceDescription = ''" />
            </VCol>
            <VCol cols="12" sm="4">
              <AppTextField type="number" :requiredField="true" :rules="[requiredValidator, greaterThanZeroValidator]"
                label="Cantidád de servicios" v-model="form.serviceDate" clearable :maxlength="15" counter
                :errorMessages="errorsBack.serviceDate" @input="errorsBack.serviceDate = ''" />
            </VCol>
            <VCol cols="12" sm="4">
              <AppTextField :requiredField="true" :rules="[requiredValidator]" label="Valor unitario"
                v-model="form.serviceValue" clearable :maxlength="15" counter :errorMessages="errorsBack.serviceValue"
                @input="errorsBack.serviceValue = ''" />
            </VCol>
            <VCol cols="12" sm="4">
              <AppTextField :requiredField="true" :rules="[requiredValidator]" label="Valor total facturado"
                v-model="form.totalFactoryValue" clearable :maxlength="15" counter
                :errorMessages="errorsBack.totalFactoryValue" @input="errorsBack.totalFactoryValue = ''" />
            </VCol>
            <VCol cols="12" sm="4">
              <AppTextField :requiredField="true" :rules="[requiredValidator]" label="Valor total reclamado"
                v-model="form.totalClaimedValue" clearable :maxlength="15" counter
                :errorMessages="errorsBack.totalClaimedValue" @input="errorsBack.totalClaimedValue = ''" />
            </VCol>
            <VCol cols="12" sm="4">
              <AppTextField :requiredField="true" :rules="[requiredValidator]"
                label="Dirección de residencia de la víctima" v-model="form.victimResidenceAddress" clearable
                :maxlength="40" counter :errorMessages="errorsBack.victimResidenceAddress"
                @input="errorsBack.victimResidenceAddress = ''" />
            </VCol>
          </VRow>
        </VForm>
      </VCardText>

      <VCardText class="d-flex justify-end gap-3 flex-wrap mt-5">
        <BtnBack :disabled="isLoading" :loading="isLoading" />
        <VBtn v-if="!disabledFiledsView" :disabled="isLoading" :loading="isLoading" @click="submitForm()">
          Guardar
        </VBtn>
      </VCardText>
    </VCard>
  </div>
</template>
