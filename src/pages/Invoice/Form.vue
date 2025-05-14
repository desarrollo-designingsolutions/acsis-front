<script lang="ts" setup>
import { useToast } from '@/composables/useToast';
import IErrorsBack from "@/interfaces/Axios/IErrorsBack";
import ModalListInvoicePayment from "@/pages/InvoicePayment/Components/ModalList.vue";
import ListService from "@/pages/Service/Components/List.vue";
import { router } from '@/plugins/1.router';
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
const serviceVendors_arrayInfo = ref([])
const entities_arrayInfo = ref([])
const tipoNotas_arrayInfo = ref([])
const patients_arrayInfo = ref([])
const statusInvoiceEnum_arrayInfo = ref([])

const form = ref({
  id: null as string | null,
  company_id: null as string | null,
  service_vendor_id: null as string | null,
  entity_id: null as string | null,
  patient_id: null as string | null,
  tipo_nota_id: null as string | null,
  invoice_number: null as string | null,
  note_number: null as string | null,
  radication_number: null as string | null,
  value_glosa: null as string | null,
  type: null as string | null,
  value_paid: null as string | null,
  remaining_balance: null as string | null,
  total: null as string | null,
  invoice_date: null as string | null,
  radication_date: null as string | null,
  status: null as string | null,
})

const soat = ref({
  id: null as string | null,
  policy_number: null as string | null,
  accident_date: null as string | null,
  start_date: null as string | null,
  end_date: null as string | null,
})

const totalValueGlosa = ref<string>('')
const totalValuePaid = ref<string>('')
const totalValueTotal = ref<string>('')
const totalValueRemainingBalance = ref<string>('')

const clearForm = () => {
  for (const key in form.value) {
    form.value[key] = null
  }
}

const fetchDataForm = async () => {

  form.value.id = route.params.id || null

  const url = form.value.id ? `/invoice/${form.value.id}/edit` : `/invoice/create`

  loading.form = true
  const { response, data } = await useAxios(url).get({
    params: {
      company_id: authenticationStore.company.id,
    }
  });
  loading.form = false


  if (response.status == 200 && data) {

    serviceVendors_arrayInfo.value = data.serviceVendors_arrayInfo
    entities_arrayInfo.value = data.entities_arrayInfo
    tipoNotas_arrayInfo.value = data.tipoNotas_arrayInfo
    patients_arrayInfo.value = data.patients_arrayInfo
    statusInvoiceEnum_arrayInfo.value = data.statusInvoiceEnum_arrayInfo

    //formulario 
    if (data.form) {
      form.value = cloneObject(data.form)

      if (form.value.type == 'INVOICE_TYPE_002') {
        soat.value = cloneObject(data?.infoDataExtra)
      }

      totalValuePaid.value = currencyFormat(formatToCurrencyFormat(form.value.value_paid));
      totalValueTotal.value = currencyFormat(formatToCurrencyFormat(form.value.total));
      totalValueRemainingBalance.value = currencyFormat(formatToCurrencyFormat(form.value.remaining_balance));

      totalValueGlosa.value = form.value.value_glosa;
      form.value.value_glosa = currencyFormat(formatToCurrencyFormat(totalValueGlosa.value));
      dataCalculate.real_value_glosa = cloneObject(totalValueGlosa.value);

      listenForInvoiceUpdates();

    }
  }
}

const submitForm = async () => {
  const validation = await formValidation.value?.validate()

  if (!isObject(form.value.patient_id)) {
    errorsBack.value.patient_id = 'El campo es obligatorio'
    return false
  }

  if (validation?.valid) {

    const url = form.value.id ? `/invoice/update/${form.value.id}` : `/invoice/store`

    form.value.company_id = authenticationStore.company.id;

    form.value.type = route.params.type;

    let document: any = [];

    switch (form.value.type) {
      case 'INVOICE_TYPE_002':
        document = { soat: soat.value };
        break;

      default:
        break;
    }

    loading.form = true;
    const { data, response } = await useAxios(url).post({
      ...form.value,
      value_glosa: dataCalculate.real_value_glosa,
      ...document,
    });

    if (response.status == 200 && data) {

      form.value.id = data.form.id

      if (form.value.type == 'INVOICE_TYPE_002') {
        soat.value.id = data.infoDataExtra.id
      }

      router.replace({ name: "Invoice-Form", params: { type: data.form.type, id: data.form.id } });

      listenForInvoiceUpdates()
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

const paramsSelectInfinite = {
  company_id: authenticationStore.company.id,
}

//FORMATO COMPONENTE MONEDA
const dataCalculate = reactive({
  real_value_glosa: 0 as number,
})

const dataReal = (data: any, field: string) => {
  dataCalculate[field] = data
}

//ModalMassUpload
const refModalMassUpload = ref()

const openModalMassUpload = () => {
  refModalMassUpload.value.openModal(form.value.id, 'Invoice');
}

//ModalShowFiles
const refModalShowFiles = ref()

const openModalShowFiles = () => {
  refModalShowFiles.value.openModal(form.value.id, 'Invoice')
}

//ModalListInvoicePayment
const refModalListInvoicePayment = ref()

const openModalListInvoicePayment = () => {
  refModalListInvoicePayment.value.openModal({ invoice_id: form.value.id })
}

const checkInvoiceNumber = async () => {

  if (form.value.invoice_number != null) {
    const url = '/invoice/validateInvoiceNumber'

    const { response, data } = await useAxios(url).post({
      invoice_number: form.value.invoice_number,
      company_id: authenticationStore.company.id,
    });

    if (response.status == 200 && data) {
      if (data.exists) {
        errorsBack.value.invoice_number = data.message_invoice;
      }
    } else {
      errorsBack.value.invoice_number = "";
    }
  }
};

const listenForInvoiceUpdates = () => {
  if (form.value.id) {

    window.Echo
      .channel(`invoice.${form.value.id}`)
      .listen('InvoiceRowUpdatedNow', (event: any) => {

        totalValueGlosa.value = currencyFormat(formatToCurrencyFormat(event.value_glosa));
        totalValueTotal.value = currencyFormat(formatToCurrencyFormat(event.total));
        totalValuePaid.value = currencyFormat(formatToCurrencyFormat(event.value_paid));
        totalValueRemainingBalance.value = currencyFormat(formatToCurrencyFormat(event.remaining_balance));
      });
  }
};

const stopListening = () => {
  if (form.value.id) {
    window.Echo.leave(`invoice.${form.value.id}`);
  }
};

onUnmounted(() => {
  stopListening();
});


const downloadJson = async () => {
  try {
    loading.form = true;

    // Hacer la solicitud GET al endpoint
    const response = await useAxios(`/invoice/downloadJson/${form.value.id}`).get({
      responseType: 'blob', // Indicar que esperamos un archivo binario
    });

    // Crear un Blob a partir de la respuesta
    const blob = new Blob([response.data], { type: 'application/json' });

    // Crear un enlace temporal para la descarga
    const url = window.URL.createObjectURL(blob);
    const link = document.createElement('a');
    link.href = url;
    link.setAttribute('download', `${form.value.invoice_number}.json`); // Nombre del archivo
    document.body.appendChild(link);
    link.click();

    // Limpiar
    document.body.removeChild(link);
    window.URL.revokeObjectURL(url);

    loading.form = false;
  } catch (error) {
    console.error('Error al descargar el archivo:', error);
    loading.form = false;
  }
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
            <VCol cols="12">
              <AppCardActions title="Información de la Factura" action-collapsed>
                <VCardText>
                  <VRow>
                    <VCol cols="12" sm="4">
                      <AppSelectRemote :disabled="disabledFiledsView" label="Prestador" v-model="form.service_vendor_id"
                        url="/selectInfiniteServiceVendor" arrayInfo="serviceVendors" :requiredField="true"
                        :rules="[requiredValidator]" clearable :params="paramsSelectInfinite"
                        :itemsData="serviceVendors_arrayInfo" :firstFetch="false">
                      </AppSelectRemote>
                    </VCol>

                    <VCol cols="12" sm="4">
                      <AppSelectRemote :disabled="disabledFiledsView" label="Entidad" v-model="form.entity_id"
                        url="/selectInfiniteEntities" arrayInfo="entities" :requiredField="true"
                        :rules="[requiredValidator]" clearable :params="paramsSelectInfinite"
                        :itemsData="entities_arrayInfo" :firstFetch="false">
                      </AppSelectRemote>
                    </VCol>

                    <VCol cols="12" sm="4">
                      <SelectPatientForm :disabled="disabledFiledsView" :requiredField="true" label="Paciente"
                        v-model="form.patient_id" :itemsData="patients_arrayInfo" :firstFetch="false"
                        :error-messages="errorsBack.patient_id" @update:modelValue="errorsBack.patient_id = ''" />
                    </VCol>

                    <VCol cols="12" sm="4">
                      <AppSelectRemote :disabled="disabledFiledsView" label="Tipo Nota" v-model="form.tipo_nota_id"
                        url="/selectInfinitetipoNota" arrayInfo="tipoNotas" clearable :params="paramsSelectInfinite"
                        :itemsData="tipoNotas_arrayInfo" :firstFetch="false" :error-messages="errorsBack.note_number"
                        @input="errorsBack.note_number = ''">
                      </AppSelectRemote>
                    </VCol>

                    <VCol cols="12" sm="4">
                      <AppTextField v-model="form.note_number" label="Número de Nota"
                        :error-messages="errorsBack.note_number" @input="errorsBack.note_number = ''" clearable />
                    </VCol>

                    <VCol cols="12" sm="4">
                      <AppSelect :items="statusInvoiceEnum_arrayInfo" v-model="form.status" label="Estado"
                        :error-messages="errorsBack.note_number" @input="errorsBack.note_number = ''" clearable
                        :requiredField="true" :rules="[requiredValidator]"></AppSelect>
                    </VCol>


                    <VCol cols="12" sm="3">
                      <AppDateTimePicker clearable :requiredField="true" label="Fecha Factura"
                        v-model="form.invoice_date" :error-messages="errorsBack.invoice_date"
                        :rules="[requiredValidator]" :config="{ dateFormat: 'Y-m-d' }" />
                    </VCol>

                    <VCol cols="12" sm="3">
                      <AppTextField @blur="checkInvoiceNumber" :requiredField="true" :rules="[requiredValidator]"
                        v-model="form.invoice_number" label="Número de Factura"
                        :error-messages="errorsBack.invoice_number" @input="errorsBack.invoice_number = ''" clearable />
                    </VCol>

                    <VCol cols="12" sm="3">
                      <AppDateTimePicker clearable :requiredField="true" label="Fecha Radicación"
                        v-model="form.radication_date" :error-messages="errorsBack.radication_date"
                        :rules="[requiredValidator]" :config="{ dateFormat: 'Y-m-d' }" />
                    </VCol>

                    <VCol cols="12" sm="3">
                      <AppTextField :requiredField="true" :rules="[requiredValidator]" v-model="form.radication_number"
                        label="Número de Radicado" :error-messages="errorsBack.radication_number"
                        @input="errorsBack.radication_number = ''" clearable />
                    </VCol>




                    <VCol cols="12" sm="3">
                      <FormatCurrency disabled label="Valor Glosado" v-model="totalValueGlosa"
                        @realValue="dataReal($event, 'real_value_glosa')" clearable />
                    </VCol>

                    <VCol cols="12" sm="3">
                      <FormatCurrency disabled label="Valor Factura" v-model="totalValueTotal" />
                    </VCol>
                    <VCol cols="12" sm="3">
                      <FormatCurrency disabled label="Valor Pagado" v-model="totalValuePaid" />
                    </VCol>

                    <VCol cols="12" sm="3">
                      <FormatCurrency disabled label="Valor restante" v-model="totalValueRemainingBalance" />
                    </VCol>

                  </VRow>
                </VCardText>
              </AppCardActions>
            </VCol>
          </VRow>

          <VRow v-if="route.params.type == 'INVOICE_TYPE_002'">
            <VCol cols="12">
              <AppCardActions title="Información SOAT" action-collapsed>
                <VCardText>
                  <VRow>
                    <VCol cols="12" sm="4">
                      <AppTextField :requiredField="true" :rules="[requiredValidator]" v-model="soat.policy_number"
                        label="Número de Póliza" :error-messages="errorsBack.policy_number"
                        @input="errorsBack.policy_number = ''" clearable />
                    </VCol>

                    <VCol cols="12" sm="4">
                      <AppDateTimePicker clearable :requiredField="true" label="Fecha Siniestro"
                        v-model="soat.accident_date" :error-messages="errorsBack.accident_date"
                        :rules="[requiredValidator]" :config="{ dateFormat: 'Y-m-d' }" />
                    </VCol>

                    <VCol cols="12" sm="4">
                      <AppDateTimePicker clearable :requiredField="true" label="Fecha Inicio SOAT"
                        v-model="soat.start_date" :error-messages="errorsBack.start_date" :rules="[requiredValidator]"
                        :config="{ dateFormat: 'Y-m-d' }" />
                    </VCol>

                    <VCol cols="12" sm="4">
                      <AppDateTimePicker clearable :requiredField="true" label="Fecha Final SOAT"
                        v-model="soat.end_date" :error-messages="errorsBack.end_date" :rules="[requiredValidator]"
                        :config="{ dateFormat: 'Y-m-d' }" />
                    </VCol>
                  </VRow>
                </VCardText>
              </AppCardActions>
            </VCol>
          </VRow>

        </VForm>
      </VCardText>

      <VCardText class="d-flex justify-end gap-3 flex-wrap mt-5">
        <VBtn v-if="form.id" color="primary" append-icon="tabler-chevron-down">
          Más Acciones
          <VMenu activator="parent">
            <VList>
              <VListItem @click="downloadJson()">
                Descargar Json
              </VListItem>
              <VListItem @click="openModalListInvoicePayment()">
                Pagos
              </VListItem>
              <VListItem @click="openModalMassUpload()">
                Añadir Soportes
              </VListItem>
              <VListItem @click="openModalShowFiles">
                Listar Soportes
              </VListItem>
            </VList>
          </VMenu>
        </VBtn>
        <BtnBack :disabled="isLoading" :loading="isLoading" />
        <VBtn v-if="!disabledFiledsView" :disabled="isLoading" :loading="isLoading" @click="submitForm()"
          color="primary">
          Guardar
        </VBtn>
      </VCardText>
    </VCard>

    <div v-if="form.id" class="mt-5">
      <ListService :invoice_id="form.id" />
    </div>

    <ModalQuestion ref="refModalQuestion" />

    <ModalMassUpload ref="refModalMassUpload" />

    <ModalShowFiles ref="refModalShowFiles"></ModalShowFiles>

    <ModalListInvoicePayment ref="refModalListInvoicePayment"></ModalListInvoicePayment>

  </div>
</template>
