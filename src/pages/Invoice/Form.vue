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
  total: null as string | null,
  invoice_date: null as string | null,
  radication_date: null as string | null,
})

const soat = ref({
  policy_number: null as string | null,
  accident_date: null as string | null,
  start_date: null as string | null,
  end_date: null as string | null,
})

const totalValueGlosa = ref<string>('')
const totalValueApproved = ref<string>('')
const totalValueTotal = ref<string>('')

const clearForm = () => {
  for (const key in form.value) {
    form.value[key] = null
  }
}

const fetchDataForm = async () => {

  form.value.id = route.params.id || null

  const url = form.value.id ? `/invoice/${route.params.type}/${form.value.id}/edit` : `/invoice/${route.params.type}/create`

  loading.form = true
  const { response, data } = await useAxios(url).get();

  if (response.status == 200 && data) {
    //formulario 
    if (data.form) {
      form.value = cloneObject(data.form)
      soat.value = cloneObject(data?.soat)

      totalValueGlosa.value = form.value.value_glosa;
      totalValueApproved.value = form.value.value_paid;
      totalValueTotal.value = form.value.total;

      form.value.value_glosa = currencyFormat(
        formatToCurrencyFormat(totalValueGlosa.value)
      );
      form.value.value_paid = currencyFormat(
        formatToCurrencyFormat(totalValueApproved.value)
      );
      form.value.total = currencyFormat(
        formatToCurrencyFormat(totalValueTotal.value)
      );
      dataCalculate.real_value_glosa = cloneObject(totalValueGlosa.value);
      dataCalculate.real_value_paid = cloneObject(totalValueApproved.value);
      dataCalculate.real_total = cloneObject(totalValueTotal.value);
    }
  }
  loading.form = false
}

const submitForm = async () => {
  const validation = await formValidation.value?.validate()
  if (validation?.valid) {

    const url = form.value.id ? `/invoice/${route.params.type}/update/${form.value.id}` : `/invoice/${route.params.type}/store`

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
      value_paid: dataCalculate.real_value_paid,
      total: dataCalculate.real_total,
      ...document,
    });

    if (response.status == 200 && data) {

      if (data.code == 200) {
        router.push({ name: 'Invoice-List' })
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

  form.value.total = '0,00';
  form.value.value_paid = '0,00';

  if (route.params.id) {
    await fetchDataForm()
  }
})

// Computed que verifica si al menos uno de los valores es true
const isLoading = computed(() => {
  return Object.values(loading).some(value => value);
});

const company = {
  company_id: authenticationStore.company.id,
}

//FORMATO COMPONENTE MONEDA
const dataCalculate = reactive({
  real_value_glosa: 0 as number,
  real_value_paid: 0 as number,
  real_total: 0 as number,
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
  if (form.value.invoice_number) {
    const url = '/invoice/validateInvoiceNumber'

    const { response, data } = await useAxios(url).post({
      invoice_number: form.value.invoice_number,
      company_id: authenticationStore.company.id,
    });

    if (response.status == 200 && data) {
      errorsBack.value.invoice_number = data.message_licences;
    } else {
      errorsBack.value.invoice_number = "";
    }
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
                      <AppSelectRemote label="Prestador" v-model="form.service_vendor_id"
                        url="/selectInfiniteServiceVendor" arrayInfo="serviceVendors" :requiredField="true"
                        :rules="[requiredValidator]" clearable :params="company">
                      </AppSelectRemote>
                    </VCol>

                    <VCol cols="12" sm="4">
                      <AppSelectRemote label="Entidad" v-model="form.entity_id" url="/selectInfiniteEntities"
                        arrayInfo="entities" :requiredField="true" :rules="[requiredValidator]" clearable
                        :params="company">
                      </AppSelectRemote>
                    </VCol>

                    <VCol cols="12" sm="4">
                      <AppSelectRemote label="Paciente" v-model="form.patient_id" url="/selectInfinitePatients"
                        arrayInfo="patients" :requiredField="true" :rules="[requiredValidator]" clearable
                        :params="company">
                      </AppSelectRemote>
                    </VCol>

                    <VCol cols="12" sm="4">
                      <AppSelectRemote label="Tipo Nota" v-model="form.tipo_nota_id" url="/selectInfinitetipoNota"
                        arrayInfo="tipoNotas" :requiredField="true" :rules="[requiredValidator]" clearable
                        :params="company">
                      </AppSelectRemote>
                    </VCol>

                    <VCol cols="12" sm="4">
                      <AppTextField :requiredField="true" :rules="[requiredValidator]" v-model="form.note_number"
                        label="Número de Nota" :error-messages="errorsBack.note_number"
                        @input="errorsBack.note_number = ''" clearable />
                    </VCol>

                    <VCol cols="12" sm="4">
                      <AppTextField @blur="checkInvoiceNumber" :requiredField="true" :rules="[requiredValidator]"
                        v-model="form.invoice_number" label="Número de Factura"
                        :error-messages="errorsBack.invoice_number" @input="errorsBack.invoice_number = ''" clearable />
                    </VCol>

                    <VCol cols="12" sm="4">
                      <AppTextField :requiredField="true" :rules="[requiredValidator]" v-model="form.radication_number"
                        label="Número de Radicado" :error-messages="errorsBack.radication_number"
                        @input="errorsBack.radication_number = ''" clearable />
                    </VCol>

                    <VCol cols="12" sm="4">
                      <AppDateTimePicker clearable :requiredField="true" label="Fecha Factura"
                        v-model="form.invoice_date" :error-messages="errorsBack.invoice_date"
                        :rules="[requiredValidator]" :config="{ dateFormat: 'Y-m-d' }" />
                    </VCol>

                    <VCol cols="12" sm="4">
                      <AppDateTimePicker clearable :requiredField="true" label="Fecha Radicación"
                        v-model="form.radication_date" :error-messages="errorsBack.radication_date"
                        :rules="[requiredValidator]" :config="{ dateFormat: 'Y-m-d' }" />
                    </VCol>

                    <VCol cols="12" sm="4">
                      <FormatCurrency :requiredField="true" :disabled="disabledFiledsView" label="Valor Glosado"
                        :rules="[requiredValidator]" v-model="form.value_glosa"
                        @realValue="dataReal($event, 'real_value_glosa')" :error-messages="errorsBack.value_glosa"
                        @input="errorsBack.value_glosa = ''" clearable />
                    </VCol>

                    <VCol cols="12" sm="4">
                      <FormatCurrency disabled label="Valor Pagado" v-model="form.value_paid"
                        @realValue="dataReal($event, 'real_value_paid')" />
                    </VCol>

                    <VCol cols="12" sm="4">
                      <FormatCurrency disabled label="Valor Factura" v-model="form.total"
                        @realValue="dataReal($event, 'real_total')" />
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
