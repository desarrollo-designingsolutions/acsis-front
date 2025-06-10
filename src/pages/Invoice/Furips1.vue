<script setup lang="ts">
import { useToast } from '@/composables/useToast';
import IErrorsBack from '@/interfaces/Axios/IErrorsBack';
import { useAuthenticationStore } from "@/stores/useAuthenticationStore";
import { reactive, ref } from 'vue';
import type { VForm } from 'vuetify/components';
const { toast } = useToast()

definePage({
  path: 'invoice-Furips1/:invoice_id/:id?',
  name: 'Invoice-Furips1',
  meta: {
    redirectIfLoggedIn: true,
    requiresAuth: true,
    requiredPermission: 'invoice.furips1',
  },
});

interface IInvoiceData {
  id: string | null;
  insurance_statuse_code: string | null;
}
interface ISelect {
  title: string;
  value: string;
}
interface IForm {
  id: null | string;
  invoice_id: null | string;
  previousFilingNumber: null | string;
  rgoResponse: null | string;
  consecutiveClaimNumber: null | string;
  victimResidenceAddress: null | string;
  victimPhone: null | string;
  victimCondition: null | string;
  eventNature: null | string;
  otherEventDescription: null | string;
  eventOccurrenceAddress: null | string;
  eventOccurrenceDate: null | string;
  eventOccurrenceTime: null | string;
  eventDepartmentCode: null | string;
  eventMunicipalityCode: null | string;
  eventZone: null | string;
  referenceType: null | string;
  referralDate: null | string;
  departureTime: null | string;
  referringHealthProviderCode: null | string;
  referringProfessional: null | string;
  referringPersonPosition: null | string;
  admissionDate: null | string;
  admissionTime: null | string;
  receivingHealthProviderCode: null | string;
  receivingProfessional: null | string;
  interinstitutionalTransferAmbulancePlate: null | string;
  vehicleBrand: null | string;
  vehiclePlate: null | string;
  vehicleType: null | string;
  sirasFilingNumber: null | string;
  insurerCapExhaustionCharge: null | string;
  surgicalProcedureComplexity: null | string;
  ownerDocumentType: null | string;
  ownerDocumentNumber: null | string;
  ownerFirstLastName: null | string;
  ownerSecondLastName: null | string;
  ownerFirstName: null | string;
  ownerSecondName: null | string;
  ownerResidenceAddress: null | string;
  ownerResidencePhone: null | string;
  ownerResidenceDepartmentCode: null | string;
  ownerResidenceMunicipalityCode: null | string;
  driverFirstLastName: null | string;
  driverSecondLastName: null | string;
  driverFirstName: null | string;
  driverSecondName: null | string;
  driverDocumentType: null | string;
  driverDocumentNumber: null | string;
  driverResidenceAddress: null | string;
  driverResidenceDepartmentCode: null | string;
  driverResidenceMunicipalityCode: null | string;
  driverResidencePhone: null | string;
  primaryTransferAmbulancePlate: null | string;
  victimTransportFromEventSite: null | string;
  victimTransportToEnd: null | string;
  transportServiceType: null | string;
  victimPickupZone: null | string;
  doctorFirstLastName: null | string;
  doctorSecondLastName: null | string;
  doctorFirstName: null | string;
  doctorSecondName: null | string;
  doctorRegistrationNumber: null | string;
  totalBilledMedicalSurgical: null | string;
  totalClaimedMedicalSurgical: null | string;
  totalBilledTransport: null | string;
  totalClaimedTransport: null | string;
  enabledServicesConfirmation: null | string;
}

const form = ref<IForm>({
  id: null,
  invoice_id: null,
  previousFilingNumber: null,
  rgoResponse: null,
  consecutiveClaimNumber: null,
  victimResidenceAddress: null,
  victimPhone: null,
  victimCondition: null,
  eventNature: null,
  otherEventDescription: null,
  eventOccurrenceAddress: null,
  eventOccurrenceDate: null,
  eventOccurrenceTime: null,
  eventDepartmentCode: null,
  eventMunicipalityCode: null,
  eventZone: null,
  referenceType: null,
  referralDate: null,
  departureTime: null,
  referringHealthProviderCode: null,
  referringProfessional: null,
  referringPersonPosition: null,
  admissionDate: null,
  admissionTime: null,
  receivingHealthProviderCode: null,
  receivingProfessional: null,
  interinstitutionalTransferAmbulancePlate: null,
  vehicleBrand: null,
  vehiclePlate: null,
  vehicleType: null,
  sirasFilingNumber: null,
  insurerCapExhaustionCharge: null,
  surgicalProcedureComplexity: null,
  ownerDocumentType: null,
  ownerDocumentNumber: null,
  ownerFirstLastName: null,
  ownerSecondLastName: null,
  ownerFirstName: null,
  ownerSecondName: null,
  ownerResidenceAddress: null,
  ownerResidencePhone: null,
  ownerResidenceDepartmentCode: null,
  ownerResidenceMunicipalityCode: null,
  driverFirstLastName: null,
  driverSecondLastName: null,
  driverFirstName: null,
  driverSecondName: null,
  driverDocumentType: null,
  driverDocumentNumber: null,
  driverResidenceAddress: null,
  driverResidenceDepartmentCode: null,
  driverResidenceMunicipalityCode: null,
  driverResidencePhone: null,
  primaryTransferAmbulancePlate: null,
  victimTransportFromEventSite: null,
  victimTransportToEnd: null,
  transportServiceType: null,
  victimPickupZone: null,
  doctorFirstLastName: null,
  doctorSecondLastName: null,
  doctorFirstName: null,
  doctorSecondName: null,
  doctorRegistrationNumber: null,
  totalBilledMedicalSurgical: null,
  totalClaimedMedicalSurgical: null,
  totalBilledTransport: null,
  totalClaimedTransport: null,
  enabledServicesConfirmation: null,
});

const clearForm = () => {
  for (const key in form.value) {
    form.value[key] = null
  }
}

const errorsBack = ref<IErrorsBack>({});
const loading = reactive({ form: false });
const isLoading = ref(false);
const disabledFiledsView = ref(false);
const route = useRoute()
const authenticationStore = useAuthenticationStore();

const tabs = ref([
  { title: 'Datos Iniciales', show: true, errorsValidations: false },
  { title: 'Información del Vehículo y Atención', show: true, errorsValidations: false },
  { title: 'Transporte, Médico y Reclamaciones', show: true, errorsValidations: false },
]);

const sections = ref([
  { title: 'I. Datos de la reclamación', show: true, errorsValidations: false },
  { title: 'III. Datos de la víctima del evento catastrófico o accidente de tránsito', show: true, errorsValidations: false },
  { title: 'IV. Datos del sitio donde ocurrió el evento catastrófico o el accidente de tránsito', show: true, errorsValidations: false },
  { title: 'IX. Datos de remisión', show: false, errorsValidations: false },
  { title: 'V. Datos del vehículo involucrado en el accidente de tránsito', show: true, errorsValidations: false },
  { title: 'VI. Datos Relacionados con la Atención de la Víctima', show: true, errorsValidations: false },
  { title: 'VII. Datos del propietario del vehículo', show: true, errorsValidations: false },
  { title: 'VIII. Datos del conductor involucrado en el accidente de tránsito', show: true, errorsValidations: false },
  { title: 'X. Transporte y movilización de la víctima', show: false, errorsValidations: false },
  { title: 'XII. Datos del médico o profesional de la salud tratante', show: true, errorsValidations: false },
  { title: 'XIII. Amparos que reclama', show: true, errorsValidations: false },
  { title: 'XIV. Confirmación servicios habilitados', show: true, errorsValidations: false },
]);

const formRefs = ref<(VForm | null)[]>(new Array(sections.value.length).fill(null));
const currentTab = ref(0);

const updateValidationToFalse = () => {
  tabs.value.forEach(tab => { tab.errorsValidations = false; });
  sections.value.forEach(section => { section.errorsValidations = false; });
};

const allValidations = async () => {
  updateValidationToFalse();
  const validations = await Promise.all(
    formRefs.value.map(async (formRef, index) => {
      if (!sections.value[index].show) return true;
      const validation = await formRef?.validate?.();
      if (validation) {
        sections.value[index].errorsValidations = !validation.valid;
        const tabIndex = Math.floor(index / 4);
        tabs.value[tabIndex].errorsValidations = tabs.value[tabIndex].errorsValidations || !validation.valid;
        return validation.valid;
      }
      return true;
    }),
  );

  const firstErrorIndex = sections.value.findIndex(section => section.errorsValidations);
  if (firstErrorIndex !== -1) {
    currentTab.value = Math.floor(firstErrorIndex / 4);
  }

  return validations.every(valid => valid === true);
};

const submitForm = async (save: boolean) => {
  isLoading.value = true;
  const isValid = await allValidations();
  if (isValid) {
    console.log('Form data:', form.value);
  } else {
    console.log('Form has errors');
  }
  isLoading.value = false;
};

// watch(form, (newForm) => {
//   sections.value[3].show = !!newForm.referralDate;
//   sections.value[8].show = !!newForm.primaryTransferAmbulancePlate;
// }, { deep: true });


const limitNumberLength = () => {
  errorsBack.value.consecutiveClaimNumber = ''

  const maxLength = 12;
  if (form.value.consecutiveClaimNumber && form.value.consecutiveClaimNumber.toString().length > maxLength) {
    form.value.consecutiveClaimNumber = form.value.consecutiveClaimNumber.toString().slice(0, maxLength);
  }
};



const rgoResponseEnum_arrayInfo = ref<ISelect[]>([])
const victimConditionEnum_arrayInfo = ref<ISelect[]>([])
const eventNatureEnum_arrayInfo = ref<ISelect[]>([])
const eventZoneEnum_arrayInfo = ref<ISelect[]>([])
const referenceTypeEnum_arrayInfo = ref<ISelect[]>([])
const ipsCodHabilitacion_arrayInfo = ref<ISelect[]>([])
const vehicleTypeEnum_arrayInfo = ref<ISelect[]>([])
const yesNoEnum_arrayInfo = ref<ISelect[]>([])
const surgicalComplexityEnum_arrayInfo = ref<ISelect[]>([])
const ownerDocumentType_arrayInfo = ref<ISelect[]>([])
const driverDocumentType_arrayInfo = ref<ISelect[]>([])
const transportServiceTypeEnum_arrayInfo = ref<ISelect[]>([])
const pickupZoneEnum_arrayInfo = ref<ISelect[]>([])

const invoice = ref<IInvoiceData>({
  id: null,
  insurance_statuse_code: null,
})

const fetchDataForm = async () => {

  form.value.invoice_id = route.params.invoice_id || null
  form.value.id = route.params.id || null

  const url = form.value.id ? `/furips1/${form.value.id}/edit` : `/furips1/create/${form.value.invoice_id}`

  loading.form = true
  const { response, data } = await useAxios(url).get();
  loading.form = false


  if (response.status == 200 && data) {

    rgoResponseEnum_arrayInfo.value = data.rgoResponseEnum_arrayInfo
    victimConditionEnum_arrayInfo.value = data.victimConditionEnum_arrayInfo
    eventNatureEnum_arrayInfo.value = data.eventNatureEnum_arrayInfo
    eventZoneEnum_arrayInfo.value = data.eventZoneEnum_arrayInfo
    referenceTypeEnum_arrayInfo.value = data.referenceTypeEnum_arrayInfo
    ipsCodHabilitacion_arrayInfo.value = data.ipsCodHabilitacion_arrayInfo
    vehicleTypeEnum_arrayInfo.value = data.vehicleTypeEnum_arrayInfo
    yesNoEnum_arrayInfo.value = data.yesNoEnum_arrayInfo
    surgicalComplexityEnum_arrayInfo.value = data.surgicalComplexityEnum_arrayInfo
    ownerDocumentType_arrayInfo.value = data.ownerDocumentType_arrayInfo
    driverDocumentType_arrayInfo.value = data.driverDocumentType_arrayInfo
    transportServiceTypeEnum_arrayInfo.value = data.transportServiceTypeEnum_arrayInfo
    pickupZoneEnum_arrayInfo.value = data.pickupZoneEnum_arrayInfo

    invoice.value = data.invoice

    //formulario 
    if (data.form) {
      form.value = cloneObject(data.form)
    }
  }
}

onMounted(async () => {
  clearForm()

  await fetchDataForm()
})


//Validations
const vehicleBrand_validation = computed(() => {
  const rules = [
    value => requiredValidator(value),
  ]

  if (invoice.value && ["5", "3"].includes(invoice.value.insurance_statuse_code ?? "")) {
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
const vehiclePlate_validation = computed(() => {
  const rules = [
    value => requiredValidator(value),
  ]

  if (invoice.value && ["1", "2", "4", "5", "6", "7"].includes(invoice.value.insurance_statuse_code ?? "")) {
    return {
      rules: rules,
      requiredField: true
    }
  }
  return {
    rules: [],
    requiredField: false
  }
})
const vehicleType_validation = computed(() => {
  const rules = [
    value => requiredValidator(value),
  ]

  if (invoice.value && ["1", "2", "4", "5", "6", "7"].includes(invoice.value.insurance_statuse_code ?? "")) {
    return {
      rules: rules,
      requiredField: true
    }
  }
  return {
    rules: [],
    requiredField: false
  }
})

const sirasFilingNumber_validation = computed(() => {
  const rules = [
    value => requiredValidator(value),
  ]

  if (["01"].includes(form.value.eventNature ?? "")) {
    return {
      rules: rules,
      requiredField: true
    }
  }
  return {
    rules: [],
    requiredField: false
  }
})

const insurerCapExhaustionCharge_validation = computed(() => {
  const rules = [
    value => requiredValidator(value),
  ]

  if (invoice.value && ["6"].includes(invoice.value.insurance_statuse_code ?? "")) {
    return {
      rules: rules,
      requiredField: true
    }
  }
  return {
    rules: [],
    requiredField: false
  }
})

const ownerDocumentType_validation = computed(() => {
  const rules = [
    value => requiredValidator(value),
  ]

  if (invoice.value && ["1", "2", "4", "6"].includes(invoice.value.insurance_statuse_code ?? "")) {
    return {
      rules: rules,
      requiredField: true
    }
  }
  return {
    rules: [],
    requiredField: false
  }
})

const dynamicDocumentLengthRule = computed(() => (value: string) => {
  const tipoId = form.value.ownerDocumentType?.codigo;

  if (!tipoId || !value) return true;
  const maxLength = documentLengthByType[tipoId] || 20;
  return (
    value.length <= maxLength ||
    `El documento ${tipoId} debe tener máximo ${maxLength} caracteres`
  );
});

const ownerDocumentNumber_validation = computed(() => {
  const rules = [
    value => requiredValidator(value),
    dynamicDocumentLengthRule.value,
  ]

  if (invoice.value && ["1", "2", "4", "6"].includes(invoice.value.insurance_statuse_code ?? "")) {
    return {
      rules: rules,
      requiredField: true
    }
  }
  return {
    rules: [dynamicDocumentLengthRule.value],
    requiredField: false
  }
})

const insuranceStatuseCode1246_validation = computed(() => {
  const rules = [
    value => requiredValidator(value),
  ]

  if (invoice.value && ["1", "2", "4", "6"].includes(invoice.value.insurance_statuse_code ?? "")) {
    return {
      rules: rules,
      requiredField: true
    }
  }
  return {
    rules: [],
    requiredField: false
  }
})
const insuranceStatuseCode12467_validation = computed(() => {
  const rules = [
    value => requiredValidator(value),
  ]

  if (invoice.value && ["1", "2", "4", "6", "7"].includes(invoice.value.insurance_statuse_code ?? "")) {
    return {
      rules: rules,
      requiredField: true
    }
  }
  return {
    rules: [],
    requiredField: false
  }
})



const driverDocumentTypeRuleMaxCharacters = computed(() => (value: string) => {
  const tipoId = form.value.driverDocumentType?.codigo;

  if (!tipoId || !value) return true;
  const maxLength = documentLengthByType[tipoId] || 20;
  return (
    value.length <= maxLength ||
    `El documento ${tipoId} debe tener máximo ${maxLength} caracteres`
  );
});

const driverDocumentType_validation = computed(() => {
  const rules = [
    value => requiredValidator(value),
    driverDocumentTypeRuleMaxCharacters.value,
  ]

  if (invoice.value && ["1", "2", "4", "6", "7"].includes(invoice.value.insurance_statuse_code ?? "")) {
    return {
      rules: rules,
      requiredField: true
    }
  }
  return {
    rules: [driverDocumentTypeRuleMaxCharacters.value],
    requiredField: false
  }
})

</script>




<template>
  <div>
    <VCard :disabled="loading.form" :loading="loading.form">
      <VCardTitle class="d-flex justify-space-between">
        <span>Información del vehículo</span>
      </VCardTitle>

      <VCardText>
        <VTabs v-model="currentTab">
          <VTab class="text-none" v-for="(tab, index) in tabs" :key="index" v-show="tab.show">
            <VIcon start :icon="!tab.errorsValidations ? '' : 'tabler-alert-circle-filled'"
              :color="!tab.errorsValidations ? '' : 'error'" />
            {{ tab.title }}
          </VTab>
        </VTabs>
      </VCardText>

      <VCardText>
        <!-- Tab 0: Datos Iniciales -->
        <div v-show="currentTab == 0">
          <!-- Section 0: Datos de la reclamación -->
          <VCardTitle>I. Datos de la reclamación</VCardTitle>
          <VForm :ref="el => formRefs[0] = el" @submit.prevent="() => { }" :disabled="disabledFiledsView">
            <VRow>
              <VCol cols="12" sm="4">
                <AppTextField :requiredField="true" label="Número de radicado anterior"
                  v-model="form.previousFilingNumber" clearable :maxlength="10" counter
                  :errorMessages="errorsBack.previousFilingNumber" @input="errorsBack.previousFilingNumber = ''"
                  :rules="[requiredValidator]" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppSelect label="RGO Respuesta a Glosa u objeción" v-model="form.rgoResponse" clearable
                  :errorMessages="errorsBack.rgoResponse" @input="errorsBack.rgoResponse = ''"
                  :items="rgoResponseEnum_arrayInfo" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppTextField type="number" :requiredField="true" label="Número consecutivo de la reclamación"
                  v-model="form.consecutiveClaimNumber" clearable :maxlength="12" counter
                  :errorMessages="errorsBack.consecutiveClaimNumber" :rules="[requiredValidator]"
                  @input="limitNumberLength" />
              </VCol>
            </VRow>
          </VForm>

          <!-- Section 1: Datos de la víctima -->

          <VCardTitle class="mt-4">III. Datos de la víctima del evento catastrófico o accidente de tránsito</VCardTitle>

          <VForm :ref="el => formRefs[1] = el" @submit.prevent="() => { }" :disabled="disabledFiledsView">
            <VRow>
              <VCol cols="12" sm="4">
                <AppTextField :requiredField="true" label="Dirección de residencia de la víctima"
                  v-model="form.victimResidenceAddress" clearable :maxlength="40" counter
                  :errorMessages="errorsBack.victimResidenceAddress" @input="errorsBack.victimResidenceAddress = ''"
                  :rules="[requiredValidator]" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppTextField :requiredField="true" label="Teléfono de la víctima" v-model="form.victimPhone" clearable
                  :maxlength="10" counter :errorMessages="errorsBack.victimPhone" @input="errorsBack.victimPhone = ''"
                  :rules="[requiredValidator]" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppSelect :requiredField="true" label="Condición de la víctima" v-model="form.victimCondition"
                  clearable :errorMessages="errorsBack.victimCondition" @input="errorsBack.victimCondition = ''"
                  :items="victimConditionEnum_arrayInfo" :rules="[requiredValidator]" />
              </VCol>
            </VRow>
          </VForm>


          <!-- Section 2: Datos del sitio del evento -->
          <VCardTitle class="mt-4">IV. Datos del sitio donde ocurrió el evento catastrófico o el accidente de tránsito
          </VCardTitle>
          <VForm :ref="el => formRefs[2] = el" @submit.prevent="() => { }" :disabled="disabledFiledsView">
            <VRow>
              <VCol cols="12" sm="4">
                <AppSelect :requiredField="true" label="Naturaleza del evento" v-model="form.eventNature" clearable
                  :errorMessages="errorsBack.eventNature" @input="errorsBack.eventNature = ''"
                  :items="eventNatureEnum_arrayInfo" :rules="[requiredValidator]" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppTextField :requiredField="form.eventNature == '17' ? true : false"
                  label="Descripción del otro evento" v-model="form.otherEventDescription" clearable :maxlength="25"
                  counter :errorMessages="errorsBack.otherEventDescription"
                  @input="errorsBack.otherEventDescription = ''"
                  :rules="[form.eventNature == '17' ? requiredValidator : []]" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppTextField :requiredField="true" label="Dirección de ocurrencia del evento"
                  v-model="form.eventOccurrenceAddress" clearable :maxlength="40" counter
                  :errorMessages="errorsBack.eventOccurrenceAddress" @input="errorsBack.eventOccurrenceAddress = ''"
                  :rules="[requiredValidator]" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppTextField type="date" :requiredField="true" label="Fecha de ocurrencia del evento"
                  v-model="form.eventOccurrenceDate" clearable :errorMessages="errorsBack.eventOccurrenceDate"
                  @input="errorsBack.eventOccurrenceDate = ''" :rules="[requiredValidator]" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppTextField type="time" :requiredField="true" label="Hora de ocurrencia del evento"
                  v-model="form.eventOccurrenceTime" clearable :errorMessages="errorsBack.eventOccurrenceTime"
                  @input="errorsBack.eventOccurrenceTime = ''" :rules="[requiredValidator]" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppTextField :requiredField="true" label="Código del departamento de ocurrencia"
                  v-model="form.eventDepartmentCode" clearable :maxlength="6" counter
                  :errorMessages="errorsBack.eventDepartmentCode" @input="errorsBack.eventDepartmentCode = ''"
                  :rules="[requiredValidator]" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppTextField :requiredField="true" label="Código del municipio de ocurrencia"
                  v-model="form.eventMunicipalityCode" clearable :maxlength="6" counter
                  :errorMessages="errorsBack.eventMunicipalityCode" @input="errorsBack.eventMunicipalityCode = ''"
                  :rules="[requiredValidator]" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppSelect :requiredField="true" label="Zona de ocurrencia del evento" v-model="form.eventZone"
                  clearable :errorMessages="errorsBack.eventZone" @input="errorsBack.eventZone = ''"
                  :items="eventZoneEnum_arrayInfo" :rules="[requiredValidator]" />
              </VCol>
            </VRow>
          </VForm>


          <!-- Section 3: Datos de remisión -->
          <VCardTitle class="mt-4">IX. Datos de remisión</VCardTitle>
          <VForm :ref="el => formRefs[3] = el" @submit.prevent="() => { }" :disabled="disabledFiledsView">
            <VRow>
              <VCol cols="12" sm="4">
                <AppSelect :requiredField="true" label="Tipo de referencia" v-model="form.referenceType" clearable
                  :errorMessages="errorsBack.referenceType" @input="errorsBack.referenceType = ''"
                  :items="referenceTypeEnum_arrayInfo" :rules="[requiredValidator]" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppTextField type="date" :requiredField="true" label="Fecha de remisión" v-model="form.referralDate"
                  clearable :errorMessages="errorsBack.referralDate" @input="errorsBack.referralDate = ''"
                  :rules="[requiredValidator]" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppTextField type="time" :requiredField="true" label="Hora de salida" v-model="form.departureTime"
                  clearable :errorMessages="errorsBack.departureTime" @input="errorsBack.departureTime = ''"
                  :rules="[requiredValidator]" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppSelectRemote clearable label="Código de habilitación del prestador de servicios de salud remitente"
                  v-model="form.referringHealthProviderCode" :requiredField="true" :rules="[requiredValidator]"
                  :error-messages="errorsBack.referringHealthProviderCode"
                  @input="errorsBack.referringHealthProviderCode = ''" :disabled="disabledFiledsView"
                  url="/selectInfiniteIpsCodHabilitacion" array-info="ipsCodHabilitacion"
                  :itemsData="ipsCodHabilitacion_arrayInfo" :firstFetch="false" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppTextField :requiredField="true" label="Profesional que remite" v-model="form.referringProfessional"
                  clearable :maxlength="60" counter :errorMessages="errorsBack.referringProfessional"
                  @input="errorsBack.referringProfessional = ''" :rules="[requiredValidator]" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppTextField :requiredField="true" label="Cargo de la persona que remite"
                  v-model="form.referringPersonPosition" clearable :maxlength="30" counter
                  :errorMessages="errorsBack.referringPersonPosition" @input="errorsBack.referringPersonPosition = ''"
                  :rules="[requiredValidator]" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppTextField type="date" :requiredField="true" label="Fecha de ingreso" v-model="form.admissionDate"
                  clearable :errorMessages="errorsBack.admissionDate" @input="errorsBack.admissionDate = ''"
                  :rules="[requiredValidator]" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppTextField type="time" :requiredField="true" label="Hora de ingreso" v-model="form.admissionTime"
                  clearable :errorMessages="errorsBack.admissionTime" @input="errorsBack.admissionTime = ''"
                  :rules="[requiredValidator]" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppSelectRemote clearable label="Código de habilitación del prestador de servicios de salud que recibe"
                  v-model="form.receivingHealthProviderCode" :requiredField="true" :rules="[requiredValidator]"
                  :error-messages="errorsBack.receivingHealthProviderCode"
                  @input="errorsBack.receivingHealthProviderCode = ''" :disabled="disabledFiledsView"
                  url="/selectInfiniteIpsCodHabilitacion" array-info="ipsCodHabilitacion"
                  :itemsData="ipsCodHabilitacion_arrayInfo" :firstFetch="false" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppTextField :requiredField="true" label="Profesional que recibe" v-model="form.receivingProfessional"
                  clearable :maxlength="60" counter :errorMessages="errorsBack.receivingProfessional"
                  @input="errorsBack.receivingProfessional = ''" :rules="[requiredValidator]" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppTextField :requiredField="true" label="Placa ambulancia que realiza el traslado interinstitucional"
                  v-model="form.interinstitutionalTransferAmbulancePlate" clearable :maxlength="6" counter
                  :errorMessages="errorsBack.interinstitutionalTransferAmbulancePlate"
                  @input="errorsBack.interinstitutionalTransferAmbulancePlate = ''" :rules="[requiredValidator]" />
              </VCol>
            </VRow>
          </VForm>
        </div>

        <!-- Tab 1: Información del Vehículo y Atención -->
        <div v-show="currentTab == 1">
          <!-- Section 4: Datos del vehículo -->
          <VCardTitle>V. Datos del vehículo involucrado en el accidente de tránsito</VCardTitle>
          <VForm :ref="el => formRefs[4] = el" @submit.prevent="() => { }" :disabled="disabledFiledsView">
            <VRow>
              <VCol cols="12" sm="4">
                <AppTextField :requiredField="vehicleBrand_validation.requiredField" label="Marca"
                  v-model="form.vehicleBrand" clearable :maxlength="15" counter :errorMessages="errorsBack.vehicleBrand"
                  @input="errorsBack.vehicleBrand = ''" :rules="vehicleBrand_validation.rules" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppTextField :requiredField="vehiclePlate_validation.requiredField" label="Placa"
                  v-model="form.vehiclePlate" clearable :maxlength="10" counter :errorMessages="errorsBack.vehiclePlate"
                  @input="errorsBack.vehiclePlate = ''" :rules="vehiclePlate_validation.rules" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppSelect :requiredField="vehicleType_validation.requiredField" label="Tipo de Vehículo"
                  v-model="form.vehicleType" clearable :errorMessages="errorsBack.vehicleType"
                  @input="errorsBack.vehicleType = ''" :items="vehicleTypeEnum_arrayInfo"
                  :rules="vehicleType_validation.rules" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppTextField :requiredField="sirasFilingNumber_validation.requiredField"
                  label="Número de radicado SIRAS (id_atencion)" v-model="form.sirasFilingNumber" clearable
                  :maxlength="20" counter :errorMessages="errorsBack.sirasFilingNumber"
                  @input="errorsBack.sirasFilingNumber = ''" :rules="sirasFilingNumber_validation.rules" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppSelect :requiredField="insurerCapExhaustionCharge_validation.requiredField"
                  label="Cobro por agotamiento tope Aseguradora" v-model="form.insurerCapExhaustionCharge" clearable
                  :items="yesNoEnum_arrayInfo" :errorMessages="errorsBack.insurerCapExhaustionCharge"
                  @input="errorsBack.insurerCapExhaustionCharge = ''"
                  :rules="insurerCapExhaustionCharge_validation.rules" />
              </VCol>
            </VRow>
          </VForm>

          <!-- Section 5: Datos relacionados con la atención -->
          <VCardTitle class="mt-4">VI. Datos Relacionados con la Atención de la Víctima</VCardTitle>
          <VForm :ref="el => formRefs[5] = el" @submit.prevent="() => { }" :disabled="disabledFiledsView">
            <VRow>
              <VCol cols="12" sm="4">
                <AppSelect :requiredField="true" label="Complejidad del procedimiento quirúrgico"
                  v-model="form.surgicalProcedureComplexity" clearable
                  :errorMessages="errorsBack.surgicalProcedureComplexity"
                  @input="errorsBack.surgicalProcedureComplexity = ''" :items="surgicalComplexityEnum_arrayInfo"
                  :rules="[requiredValidator]" />
              </VCol>
            </VRow>
          </VForm>

          <!-- Section 6: Datos del propietario -->
          <VCardTitle class="mt-4">VII. Datos del propietario del vehículo</VCardTitle>
          <VForm :ref="el => formRefs[6] = el" @submit.prevent="() => { }" :disabled="disabledFiledsView">
            <VRow>
              <VCol cols="12" sm="4">
                <AppSelectRemote :disabled="disabledFiledsView" label="Tipo de documento de identidad del propietario"
                  v-model="form.ownerDocumentType" url="/selectInfiniteTipoIdPisis" arrayInfo="tipoIdPisis"
                  :requiredField="ownerDocumentType_validation.requiredField"
                  :errorMessages="errorsBack.ownerDocumentType" @input="errorsBack.ownerDocumentType = ''"
                  :rules="ownerDocumentType_validation.rules" clearable :itemsData="ownerDocumentType_arrayInfo"
                  :firstFetch="false" :params="{
                    codigo_in: CODS_SELECT_FORM_FURIPS1_OWNERDOCUMENTTYPE,
                  }">
                </AppSelectRemote>
              </VCol>
              <VCol cols="12" sm="4">
                <AppTextField :requiredField="ownerDocumentNumber_validation.requiredField"
                  label="Número de documento de identidad del propietario" v-model="form.ownerDocumentNumber" clearable
                  :maxlength="16" counter :errorMessages="errorsBack.ownerDocumentNumber"
                  @input="errorsBack.ownerDocumentNumber = ''" :rules="ownerDocumentNumber_validation.rules" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppTextField :requiredField="insuranceStatuseCode1246_validation.requiredField"
                  label="Primer apellido del propietario o razón social en caso de empresa"
                  v-model="form.ownerFirstLastName" clearable :maxlength="40" counter
                  :errorMessages="errorsBack.ownerFirstLastName" @input="errorsBack.ownerFirstLastName = ''"
                  :rules="insuranceStatuseCode1246_validation.rules" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppTextField label="Segundo apellido del propietario" v-model="form.ownerSecondLastName" clearable
                  :maxlength="30" counter :errorMessages="errorsBack.ownerSecondLastName"
                  @input="errorsBack.ownerSecondLastName = ''" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppTextField :requiredField="insuranceStatuseCode1246_validation.requiredField"
                  label="Primer nombre del propietario" v-model="form.ownerFirstName" clearable :maxlength="20" counter
                  :errorMessages="errorsBack.ownerFirstName" @input="errorsBack.ownerFirstName = ''"
                  :rules="insuranceStatuseCode1246_validation.rules" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppTextField label="Segundo nombre del propietario" v-model="form.ownerSecondName" clearable
                  :maxlength="30" counter :errorMessages="errorsBack.ownerSecondName"
                  @input="errorsBack.ownerSecondName = ''" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppTextField :requiredField="insuranceStatuseCode1246_validation.requiredField"
                  label="Dirección de residencia del propietario" v-model="form.ownerResidenceAddress" clearable
                  :maxlength="40" counter :errorMessages="errorsBack.ownerResidenceAddress"
                  @input="errorsBack.ownerResidenceAddress = ''" :rules="insuranceStatuseCode1246_validation.rules" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppTextField :requiredField="insuranceStatuseCode1246_validation.requiredField"
                  label="Teléfono de residencia del propietario" v-model="form.ownerResidencePhone" clearable
                  :maxlength="10" counter :errorMessages="errorsBack.ownerResidencePhone"
                  @input="errorsBack.ownerResidencePhone = ''" :rules="insuranceStatuseCode1246_validation.rules" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppTextField :requiredField="insuranceStatuseCode1246_validation.requiredField"
                  label="Código del departamento de residencia del propietario"
                  v-model="form.ownerResidenceDepartmentCode" clearable :maxlength="6" counter
                  :errorMessages="errorsBack.ownerResidenceDepartmentCode"
                  @input="errorsBack.ownerResidenceDepartmentCode = ''"
                  :rules="insuranceStatuseCode1246_validation.rules" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppTextField :requiredField="insuranceStatuseCode1246_validation.requiredField"
                  label="Código del municipio de residencia del propietario"
                  v-model="form.ownerResidenceMunicipalityCode" clearable :maxlength="6" counter
                  :errorMessages="errorsBack.ownerResidenceMunicipalityCode"
                  @input="errorsBack.ownerResidenceMunicipalityCode = ''"
                  :rules="insuranceStatuseCode1246_validation.rules" />
              </VCol>
            </VRow>
          </VForm>

          <!-- Section 7: Datos del conductor -->
          <VCardTitle class="mt-4">VIII. Datos del conductor involucrado en el accidente de tránsito</VCardTitle>
          <VForm :ref="el => formRefs[7] = el" @submit.prevent="true" :disabled="disabledFiledsView">
            <VRow>
              <VCol cols="12" sm="4">
                <AppTextField :requiredField="insuranceStatuseCode12467_validation.requiredField"
                  label="Primer apellido del conductor" v-model="form.driverFirstLastName" clearable :maxlength="20"
                  counter :errorMessages="errorsBack.driverFirstLastName" @input="errorsBack.driverFirstLastName = ''"
                  :rules="insuranceStatuseCode12467_validation.rules" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppTextField label="Segundo apellido del conductor" v-model="form.driverSecondLastName" clearable
                  :maxlength="30" counter :errorMessages="errorsBack.driverSecondLastName"
                  @input="errorsBack.driverSecondLastName = ''" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppTextField :requiredField="insuranceStatuseCode12467_validation.requiredField"
                  label="Primer nombre del conductor" v-model="form.driverFirstName" clearable :maxlength="20" counter
                  :errorMessages="errorsBack.driverFirstName" @input="errorsBack.driverFirstName = ''"
                  :rules="insuranceStatuseCode12467_validation.rules" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppTextField label="Segundo nombre del conductor" v-model="form.driverSecondName" clearable
                  :maxlength="30" counter :errorMessages="errorsBack.driverSecondName"
                  @input="errorsBack.driverSecondName = ''" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppSelectRemote :disabled="disabledFiledsView" label="Tipo de documento de identidad del conductor"
                  v-model="form.driverDocumentType" url="/selectInfiniteTipoIdPisis" arrayInfo="tipoIdPisis"
                  :requiredField="insuranceStatuseCode12467_validation.requiredField"
                  :errorMessages="errorsBack.driverDocumentType" @input="errorsBack.driverDocumentType = ''"
                  :rules="insuranceStatuseCode12467_validation.rules" clearable
                  :itemsData="driverDocumentType_arrayInfo" :firstFetch="false" :params="{
                    codigo_in: CODS_SELECT_FORM_FURIPS1_DRIVERDOCUMENTTYPE,
                  }">
                </AppSelectRemote>
              </VCol>
              <VCol cols="12" sm="4">
                <AppTextField :requiredField="driverDocumentType_validation.requiredField"
                  label="Número de documento de identidad del conductor" v-model="form.driverDocumentNumber" clearable
                  :maxlength="16" counter :errorMessages="errorsBack.driverDocumentNumber"
                  @input="errorsBack.driverDocumentNumber = ''" :rules="driverDocumentType_validation.rules" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppTextField :requiredField="insuranceStatuseCode12467_validation.requiredField"
                  label="Dirección de residencia del conductor" v-model="form.driverResidenceAddress" clearable
                  :maxlength="40" counter :errorMessages="errorsBack.driverResidenceAddress"
                  @input="errorsBack.driverResidenceAddress = ''" :rules="insuranceStatuseCode12467_validation.rules" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppTextField :requiredField="insuranceStatuseCode12467_validation.requiredField"
                  label="Código del departamento de residencia del conductor"
                  v-model="form.driverResidenceDepartmentCode" clearable :maxlength="6" counter
                  :errorMessages="errorsBack.driverResidenceDepartmentCode"
                  @input="errorsBack.driverResidenceDepartmentCode = ''"
                  :rules="insuranceStatuseCode12467_validation.rules" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppTextField :requiredField="insuranceStatuseCode12467_validation.requiredField"
                  label="Código del municipio de residencia del conductor"
                  v-model="form.driverResidenceMunicipalityCode" clearable :maxlength="6" counter
                  :errorMessages="errorsBack.driverResidenceMunicipalityCode"
                  @input="errorsBack.driverResidenceMunicipalityCode = ''"
                  :rules="insuranceStatuseCode12467_validation.rules" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppTextField :requiredField="insuranceStatuseCode12467_validation.requiredField"
                  label="Teléfono de residencia del conductor" v-model="form.driverResidencePhone" clearable
                  :maxlength="10" counter :errorMessages="errorsBack.driverResidencePhone"
                  @input="errorsBack.driverResidencePhone = ''" :rules="insuranceStatuseCode12467_validation.rules" />
              </VCol>
            </VRow>
          </VForm>
        </div>

        <!-- Tab 2: Transporte, Médico y Reclamaciones -->
        <div v-show="currentTab == 2">
          <!-- Section 8: Transporte y movilización -->
          <VCardTitle>X. Transporte y movilización de la víctima</VCardTitle>
          <VForm :ref="el => formRefs[8] = el" @submit.prevent="() => { }" :disabled="disabledFiledsView">
            <VRow>
              <VCol cols="12" sm="4">
                <AppTextField :requiredField="true" label="Placa ambulancia traslado primario"
                  v-model="form.primaryTransferAmbulancePlate" clearable :maxlength="6" counter
                  :errorMessages="errorsBack.primaryTransferAmbulancePlate"
                  @input="errorsBack.primaryTransferAmbulancePlate = ''" :rules="[requiredValidator]" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppTextField :requiredField="true" label="Transporte de la víctima desde el sitio del evento"
                  v-model="form.victimTransportFromEventSite" clearable :maxlength="40" counter
                  :errorMessages="errorsBack.victimTransportFromEventSite"
                  @input="errorsBack.victimTransportFromEventSite = ''" :rules="[requiredValidator]" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppTextField :requiredField="true" label="Transporte de la víctima hasta el fin del recorrido"
                  v-model="form.victimTransportToEnd" clearable :maxlength="40" counter
                  :errorMessages="errorsBack.victimTransportToEnd" @input="errorsBack.victimTransportToEnd = ''"
                  :rules="[requiredValidator]" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppSelect :requiredField="true" label="Tipo de servicio del transporte"
                  v-model="form.transportServiceType" clearable :errorMessages="errorsBack.transportServiceType"
                  @input="errorsBack.transportServiceType = ''" :items="transportServiceTypeEnum_arrayInfo"
                  :rules="[requiredValidator]" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppSelect :requiredField="true" label="Zona donde recoge víctima" v-model="form.victimPickupZone"
                  clearable :errorMessages="errorsBack.victimPickupZone" @input="errorsBack.victimPickupZone = ''"
                  :items="pickupZoneEnum_arrayInfo" :rules="[requiredValidator]" />
              </VCol>
            </VRow>
          </VForm>

          <!-- Section 9: Datos del médico -->
          <VCardTitle class="mt-4">XII. Datos del médico o profesional de la salud tratante</VCardTitle>
          <VForm :ref="el => formRefs[9] = el" @submit.prevent="() => { }" :disabled="disabledFiledsView">
            <VRow>
              <VCol cols="12" sm="4">
                <AppTextField :requiredField="true" label="Primer apellido del médico o profesional de la salud"
                  v-model="form.doctorFirstLastName" clearable :maxlength="20" counter
                  :errorMessages="errorsBack.doctorFirstLastName" @input="errorsBack.doctorFirstLastName = ''"
                  :rules="[requiredValidator]" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppTextField label="Segundo apellido del médico o profesional de la salud"
                  v-model="form.doctorSecondLastName" clearable :maxlength="30" counter
                  :errorMessages="errorsBack.doctorSecondLastName" @input="errorsBack.doctorSecondLastName = ''" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppTextField :requiredField="true" label="Primer nombre del médico o profesional de la salud"
                  v-model="form.doctorFirstName" clearable :maxlength="20" counter
                  :errorMessages="errorsBack.doctorFirstName" @input="errorsBack.doctorFirstName = ''"
                  :rules="[requiredValidator]" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppTextField label="Segundo nombre del médico o profesional de la salud"
                  v-model="form.doctorSecondName" clearable :maxlength="30" counter
                  :errorMessages="errorsBack.doctorSecondName" @input="errorsBack.doctorSecondName = ''" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppTextField :requiredField="true" label="Número de registro del médico"
                  v-model="form.doctorRegistrationNumber" clearable :maxlength="16" counter
                  :errorMessages="errorsBack.doctorRegistrationNumber" @input="errorsBack.doctorRegistrationNumber = ''"
                  :rules="[requiredValidator]" />
              </VCol>
            </VRow>
          </VForm>

          <!-- Section 10: Amparos que reclama -->
          <VCardTitle class="mt-4">XIII. Amparos que reclama</VCardTitle>
          <VForm :ref="el => formRefs[10] = el" @submit.prevent="() => { }" :disabled="disabledFiledsView">
            <VRow>
              <VCol cols="12" sm="4">
                <AppTextField type="number" :requiredField="true"
                  label="Total facturado por amparo de gastos médicos quirúrgicos"
                  v-model="form.totalBilledMedicalSurgical" clearable :maxlength="15" counter
                  :errorMessages="errorsBack.totalBilledMedicalSurgical"
                  @input="errorsBack.totalBilledMedicalSurgical = ''" :rules="[requiredValidator]" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppTextField type="number" :requiredField="true"
                  label="Total reclamado por amparo de gastos médicos quirúrgicos"
                  v-model="form.totalClaimedMedicalSurgical" clearable :maxlength="15" counter
                  :errorMessages="errorsBack.totalClaimedMedicalSurgical"
                  @input="errorsBack.totalClaimedMedicalSurgical = ''" :rules="[requiredValidator]" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppTextField type="number" :requiredField="true"
                  label="Total facturado por amparo de gastos de transporte y movilización de la víctima"
                  v-model="form.totalBilledTransport" clearable :maxlength="15" counter
                  :errorMessages="errorsBack.totalBilledTransport" @input="errorsBack.totalBilledTransport = ''"
                  :rules="[requiredValidator]" />
              </VCol>
              <VCol cols="12" sm="4">
                <AppTextField type="number" :requiredField="true"
                  label="Total reclamado por amparo de gastos de transporte y movilización de la víctima"
                  v-model="form.totalClaimedTransport" clearable :maxlength="15" counter
                  :errorMessages="errorsBack.totalClaimedTransport" @input="errorsBack.totalClaimedTransport = ''"
                  :rules="[requiredValidator]" />
              </VCol>
            </VRow>
          </VForm>

          <!-- Section 11: Confirmación servicios habilitados -->
          <VCardTitle class="mt-4">XIV. Confirmación servicios habilitados</VCardTitle>
          <VForm :ref="el => formRefs[11] = el" @submit.prevent="() => { }" :disabled="disabledFiledsView">
            <VRow>
              <VCol cols="12" sm="4">
                <AppSelect :requiredField="true" label="Manifestación de servicios habilitados"
                  v-model="form.enabledServicesConfirmation" clearable
                  :errorMessages="errorsBack.enabledServicesConfirmation"
                  @input="errorsBack.enabledServicesConfirmation = ''" :items="yesNoEnum_arrayInfo"
                  :rules="[requiredValidator]" />
              </VCol>
            </VRow>
          </VForm>
        </div>
      </VCardText>

      <VCardText class="d-flex justify-end gap-3 flex-wrap mt-5">
        <BtnBack :disabled="isLoading" :loading="isLoading" />
        <VBtn v-if="!disabledFiledsView" :disabled="isLoading" :loading="isLoading" @click="submitForm(true)">
          Guardar
        </VBtn>
      </VCardText>
    </VCard>
  </div>
</template>
