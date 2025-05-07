<script setup lang="ts">
import { VDataTable } from 'vuetify/labs/VDataTable';
import type { VForm } from 'vuetify/components/VForm';
import Swal from 'sweetalert2';
import { useRipStore } from '@/pages/Rips/Automatic/Store/useRipStore';

import { useRipManualStore } from '@/pages/Rips/Manual/Store/useRipManualStore';
const { toast } = useToast();
const router = useRouter()
const ripManualStore = useRipManualStore()
const {
  cupsRips_arrayInfo,
  cupsRips_countLinks,
  cie10s_arrayInfo,
  cie10s_countLinks,
  modalidadAtencion_arrayInfo,
  modalidadAtencion_countLinks,

  ripsFinalidadConsultaVersion2_arrayInfo,
  ripsFinalidadConsultaVersion2_countLinks,

  ripsCausaExternaVersion2_arrayInfo_consultas,
  ripsCausaExternaVersion2_countLinks_consultas,

  servicio_arrayInfo,
  servicio_countLinks,
  conceptoRecaudo_arrayInfo,
  conceptoRecaudo_countLinks,
  ripsTipoDiagnosticoPrincipalVersion2_arrayInfo,
  ripsTipoDiagnosticoPrincipalVersion2_countLinks,
  grupoServicio_arrayInfo,
  grupoServicio_countLinks,
} = storeToRefs(ripManualStore)

const { dataRip, servicesCount } = storeToRefs(useRipStore())
const refForm = ref<VForm>()


//table
const inputsTableFilter = ref([
  { key: "fechaInicioAtencion", title: 'Fecha Inicio Atención', sortable: false, width: "200" },
  { key: "numAutoriacion", title: 'No Autorización', sortable: false, width: "200" },
  { key: "codConsulta", title: 'Codigo de Consulta', sortable: false, width: "350" },
  { key: "modalidadGrupoServicioTecSal", title: 'Modalidad Grupo Servicio TecSal', sortable: false, width: "350" },
  { key: "grupoServicios", title: 'Grupo Servicios', sortable: false, width: "350" },
  { key: "codServicio", title: 'Codigo Servicio', sortable: false, width: "350" },
  { key: "finalidadTecnologiaSalud", title: 'Finalidad Tecnologia Salud', sortable: false, width: "350" },
  { key: "causaMotivoAtencion", title: 'Causa Motivo Atención', sortable: false, width: "350" },
  { key: "codDiagnosticoPrincipal", title: 'Diagnostico Principal', sortable: false, width: "350" },
  { key: "codDiagnosticoRelacionado1", title: 'Diagnostico Relacionado 1', sortable: false, width: "200" },
  { key: "codDiagnosticoRelacionado2", title: 'Diagnostico Relacionado 2', sortable: false, width: "200" },
  { key: "codDiagnosticoRelacionado3", title: 'Diagnostico Relacionado 3', sortable: false, width: "200" },
  { key: "tipoDiagnosticoPrincipal", title: 'Tipo Diagnostico Principal', sortable: false, width: "200" },
  { key: "vrServicio", title: 'Valor Servicio', sortable: false, width: "200" },
  { key: "valorPagoModerador", title: 'Valor Pago Moderador', sortable: false, width: "200" },
  { key: "conceptoRecaudo", title: 'Concepto Recaudo', sortable: false, width: "350" },
  { key: 'actions', title: 'Acciones', type: 'actions', sortable: false, width: "200" },
])


const options = ref({ page: 1, itemsPerPage: 10, sortBy: [''], sortDesc: [false] })
const search = ref('')
const dataServices = ref<Array<object>>([])
const route = useRoute()

const loading = reactive({
  table: false,
})

const cloneDataSelectsInfinites = ref({})
const selectsInfinites = {
  cupsRips: [],
  modalidadAtencion: [],
  servicio: [],
  ripsCausaExternaVersion2_consultas: [],
  ripsFinalidadConsultaVersion2: [],
  cie10s_1: [],
  cie10s_2: [],
  cie10s_3: [],
  cie10s_4: [],
  conceptoRecaudo: [],
  ripsTipoDiagnosticoPrincipalVersion2: [],
  grupoServicio: [],
}
onMounted(async () => {

  const invoice = dataRip.value?.arrayData.find(ele => ele.numFactura == route.params.numFactura)
  const user = invoice?.usuarios.find(ele => ele.numDocumentoIdentificacion == route.params.numDocumentoIdentificacion)
  const servicios = user?.servicios?.consultas ?? []

  cloneDataSelectsInfinites.value = {
    'cupsRips': {
      arrayInfo: cupsRips_arrayInfo.value,
      countLinks: cupsRips_countLinks.value
    },
    'modalidadAtencion': {
      arrayInfo: modalidadAtencion_arrayInfo.value,
      countLinks: modalidadAtencion_countLinks.value
    },
    'servicio': {
      arrayInfo: servicio_arrayInfo.value,
      countLinks: servicio_countLinks.value
    },
    'ripsCausaExternaVersion2_consultas': {
      arrayInfo: ripsCausaExternaVersion2_arrayInfo_consultas.value,
      countLinks: ripsCausaExternaVersion2_countLinks_consultas.value
    },
    'ripsFinalidadConsultaVersion2': {
      arrayInfo: ripsFinalidadConsultaVersion2_arrayInfo.value,
      countLinks: ripsFinalidadConsultaVersion2_countLinks.value
    },
    'conceptoRecaudo': {
      arrayInfo: conceptoRecaudo_arrayInfo.value,
      countLinks: conceptoRecaudo_countLinks.value
    },
    'ripsTipoDiagnosticoPrincipalVersion2': {
      arrayInfo: ripsTipoDiagnosticoPrincipalVersion2_arrayInfo.value,
      countLinks: ripsTipoDiagnosticoPrincipalVersion2_countLinks.value
    },
    'grupoServicio': {
      arrayInfo: grupoServicio_arrayInfo.value,
      countLinks: grupoServicio_countLinks.value
    },
  };

  for (let index = 1; index <= 4; index++) {
    cloneDataSelectsInfinites.value['cie10s_' + index] = {
      arrayInfo: cie10s_arrayInfo.value,
      countLinks: cie10s_countLinks.value
    };
  }


  dataServices.value = JSON.parse(JSON.stringify(servicios));

  dataServices.value.forEach(element => {
    loadDataSelectsIninites()
  });


})
const loadDataSelectsIninites = () => {
  for (const key in cloneDataSelectsInfinites.value) {

    let newVariable = null
    if (key == "cupsRips") {
      cupsRips_arrayInfo.value = cloneDataSelectsInfinites.value[key].arrayInfo
      cupsRips_countLinks.value = cloneDataSelectsInfinites.value[key].countLinks

      newVariable = null
      newVariable = useSelect(
        ripManualStore.selectCupsRips,
        cupsRips_arrayInfo,
        cupsRips_countLinks,
        {}
      );
    }
    if (key == "modalidadAtencion") {
      modalidadAtencion_arrayInfo.value = cloneDataSelectsInfinites.value[key].arrayInfo
      modalidadAtencion_countLinks.value = cloneDataSelectsInfinites.value[key].countLinks

      newVariable = null
      newVariable = useSelect(
        ripManualStore.selectModalidadAtencion,
        modalidadAtencion_arrayInfo,
        modalidadAtencion_countLinks,
        {}
      );
    }
    if (key == "servicio") {
      servicio_arrayInfo.value = cloneDataSelectsInfinites.value[key].arrayInfo
      servicio_countLinks.value = cloneDataSelectsInfinites.value[key].countLinks

      newVariable = null
      newVariable = useSelect(
        ripManualStore.selectServicio,
        servicio_arrayInfo,
        servicio_countLinks,
        {}
      );
    }
    if (key == "ripsCausaExternaVersion2_consultas") {
      ripsCausaExternaVersion2_arrayInfo_consultas.value = cloneDataSelectsInfinites.value[key].arrayInfo
      ripsCausaExternaVersion2_countLinks_consultas.value = cloneDataSelectsInfinites.value[key].countLinks

      newVariable = null
      newVariable = useSelect(
        ripManualStore.selectRipsCausaExternaVersion2_consultas,
        ripsCausaExternaVersion2_arrayInfo_consultas,
        ripsCausaExternaVersion2_countLinks_consultas,
        {
          codAllowed: [21, 22, 23, 24, 25, 26, 27, 28, 29, 30]
        }
      );
    }
    if (key == "ripsFinalidadConsultaVersion2") {
      ripsFinalidadConsultaVersion2_arrayInfo.value = cloneDataSelectsInfinites.value[key].arrayInfo
      ripsFinalidadConsultaVersion2_countLinks.value = cloneDataSelectsInfinites.value[key].countLinks

      newVariable = null
      newVariable = useSelect(
        ripManualStore.selectRipsFinalidadConsultaVersion2,
        ripsFinalidadConsultaVersion2_arrayInfo,
        ripsFinalidadConsultaVersion2_countLinks,
        {
          codAllowed: [12, 13, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 27, 43, 44]

        }
      );
    }
    if (key == "cie10s_1" || key == "cie10s_2" || key == "cie10s_3" || key == "cie10s_4") {

      cie10s_arrayInfo.value = cloneDataSelectsInfinites.value[key].arrayInfo
      cie10s_countLinks.value = cloneDataSelectsInfinites.value[key].countLinks

      newVariable = null
      newVariable = useSelect(
        ripManualStore.selectCie10s,
        cie10s_arrayInfo,
        cie10s_countLinks,
        {}
      );
    }

    if (key == "conceptoRecaudo") {
      conceptoRecaudo_arrayInfo.value = cloneDataSelectsInfinites.value[key].arrayInfo
      conceptoRecaudo_countLinks.value = cloneDataSelectsInfinites.value[key].countLinks

      newVariable = null
      newVariable = useSelect(
        ripManualStore.selectConceptoRecaudo,
        conceptoRecaudo_arrayInfo,
        conceptoRecaudo_countLinks,
        {}
      );
    }
    if (key == "ripsTipoDiagnosticoPrincipalVersion2") {
      ripsTipoDiagnosticoPrincipalVersion2_arrayInfo.value = cloneDataSelectsInfinites.value[key].arrayInfo
      ripsTipoDiagnosticoPrincipalVersion2_countLinks.value = cloneDataSelectsInfinites.value[key].countLinks

      newVariable = null
      newVariable = useSelect(
        ripManualStore.selectRipsTipoDiagnosticoPrincipalVersion2,
        ripsTipoDiagnosticoPrincipalVersion2_arrayInfo,
        ripsTipoDiagnosticoPrincipalVersion2_countLinks,
        {}
      );
    }

    if (key == "grupoServicio") {
      grupoServicio_arrayInfo.value = cloneDataSelectsInfinites.value[key].arrayInfo
      grupoServicio_countLinks.value = cloneDataSelectsInfinites.value[key].countLinks

      newVariable = null
      newVariable = useSelect(
        ripManualStore.selectGrupoServicio,
        grupoServicio_arrayInfo,
        grupoServicio_countLinks,
        {}
      );
    }

    newVariable.dataNueva.value = JSON.parse(JSON.stringify(cloneDataSelectsInfinites.value[key].arrayInfo));
    newVariable.totalLinks.value = JSON.parse(JSON.stringify(cloneDataSelectsInfinites.value[key].countLinks));

    selectsInfinites[key].push(newVariable);
  }
}



const addData = async () => {

  const validation = await refForm.value?.validate()

  if (validation?.valid) {
    dataServices.value.push({
      codPrestador: null,
      fechaInicioAtencion: '',
      numAutoriacion: null,
      codConsulta: null,
      modalidadGrupoServicioTecSal: null,
      grupoServicios: null,
      codServicio: null,
      finalidadTecnologiaSalud: null,
      causaMotivoAtencion: null,
      codDiagnosticoPrincipal: null,
      codDiagnosticoRelacionado1: null,
      codDiagnosticoRelacionado2: null,
      codDiagnosticoRelacionado3: null,
      tipoDiagnosticoPrincipal: null,
      tipoDocumentoIdentificacion: null,
      numDocumentoIdentificacion: null,
      valorPagoModerador: null,
      numFEVPagoModerador: null,
      consecutivo: null,
      vrServicio: null,
    })
    loadDataSelectsIninites()
  } else {
    toast('Faltan campos por diligenciar', '', 'warning')
  }
}
const deleteData = (index: number) => {
  dataServices.value.splice(index, 1);
};

const saveData = async () => {
  const validation = await refForm.value?.validate()

  if (validation?.valid) {
    Swal.fire({
      title: 'Está seguro que desea guardar el registro?',
      showDenyButton: true,
      showCancelButton: false,
      confirmButtonText: 'Si',
      denyButtonText: 'No',
      customClass: {
        container: 'my-swal'
      }
    }).then(async result => {
      if (result.isConfirmed) {

        loading.table = true;
        const { data, response } = await useApi(`/ripManual-storeServices`).post({
          numFactura: route.params.numFactura,
          rip_id: route.params.id,
          numDocumentoIdentificacion: route.params.numDocumentoIdentificacion,
          dataServices: dataServices.value,
          service: "consultas",
        });
        loading.table = false;

        if (response.value?.ok && data.value) {
          servicesCount.value.consultas = data.value.services.length
          dataServices.value.forEach((element, index) => {
            dataServices.value[index].consecutivo = index + 1;
          });
        }
      }
    })
  } else {
    toast('Faltan campos por diligenciar', '', 'warning')
  }
}


const goBack = () => {
  const hasUndefinedConsecutivo = dataServices.value.some(service => {
    return !service.hasOwnProperty('consecutivo') || // Si no tiene la propiedad
      service.consecutivo === null ||           // Si es null
      service.consecutivo === '' ||             // Si es vacío
      typeof service.consecutivo === 'undefined'; // Si es indefinido
  });

  if (hasUndefinedConsecutivo) {
    toast("Error", "Existen elementos que no se han guardado", "danger");
    return false
  }

  router.push({ name: 'RipUsersViewForm', params: { id: route.params.id, numFactura: route.params.numFactura } })
}

</script>
<template>
  <div>
    <VCard>
      <VCardText>
        <VRow>
          <VCol cols="12" offset-md="8" md="4">
            <div class="d-flex justify-end gap-3 flex-wrap">
              <VBtn color="primary" @click="addData()"><VIcon start icon="tabler-plus" />Agregar Consulta</VBtn>
            </div>
          </VCol>
          <VCol cols="12" offset-md="8" md="4">
            <AppTextField v-model="search" density="compact" placeholder="Search ..." append-inner-icon="tabler-search"
              single-line hide-details dense outlined />
          </VCol>
        </VRow>
      </VCardText>

      <VCardText>
        <VForm ref="refForm" @submit.prevent="() => { }">
          <VDataTable :search="search" :headers="inputsTableFilter" :items="dataServices"
            :items-per-page="options.itemsPerPage" :page="options.page" :options="options">

            <template #item.fechaInicioAtencion="{ item, index }">
              <div class="text-center">
                <AppDateTimePicker v-model="item.fechaInicioAtencion" :rules="[requiredValidator]"
                  :config="{ enableTime: true, dateFormat: 'Y-m-d H:i' }" />
              </div>
            </template>

            <template #item.numAutoriacion="{ item, index }">
              <div class="text-center">
                <AppTextField clearable v-model="item.numAutoriacion" />
              </div>
            </template>

            <template #item.codConsulta="{ item, index }">
              <div class="text-center">
                <SelectInfinite item-value="code" :key="index" v-model="item.codConsulta"
                  :select="selectsInfinites['cupsRips'][index]" clearable :rules="[requiredValidator]">
                </SelectInfinite>
              </div>
            </template>

            <template #item.modalidadGrupoServicioTecSal="{ item, index }">
              <div class="text-center">
                <SelectInfinite item-value="code" :key="index" v-model="item.modalidadGrupoServicioTecSal"
                  :select="selectsInfinites['modalidadAtencion'][index]" clearable :rules="[requiredValidator]">
                </SelectInfinite>
              </div>
            </template>

            <template #item.grupoServicios="{ item, index }">
              <div class="text-center">
                <SelectInfinite item-value="code" :key="index" v-model="item.grupoServicios"
                  :select="selectsInfinites['grupoServicio'][index]" clearable :rules="[requiredValidator]">
                </SelectInfinite>
              </div>
            </template>

            <template #item.codServicio="{ item, index }">
              <div class="text-center">
                <SelectInfinite item-value="code" :key="index" v-model="item.codServicio"
                  :select="selectsInfinites['servicio'][index]" clearable :rules="[requiredValidator]">
                </SelectInfinite>
              </div>
            </template>

            <template #item.finalidadTecnologiaSalud="{ item, index }">
              <div class="text-center">
                <SelectInfinite item-value="code" :key="index" v-model="item.finalidadTecnologiaSalud"
                  :select="selectsInfinites['ripsFinalidadConsultaVersion2'][index]" clearable
                  :rules="[requiredValidator]">
                </SelectInfinite>
              </div>
            </template>

            <template #item.causaMotivoAtencion="{ item, index }">
              <div class="text-center">
                <SelectInfinite item-value="code" :key="index" v-model="item.causaMotivoAtencion"
                  :select="selectsInfinites['ripsCausaExternaVersion2_consultas'][index]" clearable
                  :rules="[requiredValidator]">
                </SelectInfinite>
              </div>
            </template>

            <template #item.codDiagnosticoPrincipal="{ item, index }">
              <div class="text-center">
                <SelectInfinite item-value="code" :key="index" v-model="item.codDiagnosticoPrincipal"
                  :select="selectsInfinites['cie10s_1'][index]" clearable :rules="[requiredValidator]">
                </SelectInfinite>
              </div>
            </template>

            <template #item.codDiagnosticoRelacionado1="{ item, index }">
              <div class="text-center">
                <SelectInfinite item-value="code" :key="index" v-model="item.codDiagnosticoRelacionado1"
                  :select="selectsInfinites['cie10s_2'][index]" clearable :rules="[requiredValidator]">
                </SelectInfinite>
              </div>
            </template>

            <template #item.codDiagnosticoRelacionado2="{ item, index }">
              <div class="text-center">
                <SelectInfinite item-value="code" :key="index" v-model="item.codDiagnosticoRelacionado2"
                  :select="selectsInfinites['cie10s_3'][index]" clearable :rules="[requiredValidator]">
                </SelectInfinite>
              </div>
            </template>

            <template #item.codDiagnosticoRelacionado3="{ item, index }">
              <div class="text-center">
                <SelectInfinite item-value="code" :key="index" v-model="item.codDiagnosticoRelacionado3"
                  :select="selectsInfinites['cie10s_4'][index]" clearable :rules="[requiredValidator]">
                </SelectInfinite>
              </div>
            </template>

            <template #item.tipoDiagnosticoPrincipal="{ item, index }">
              <div class="text-center">
                <SelectInfinite item-value="code" :key="index" v-model="item.tipoDiagnosticoPrincipal"
                  :select="selectsInfinites['ripsTipoDiagnosticoPrincipalVersion2'][index]" clearable
                  :rules="[requiredValidator]">
                </SelectInfinite>
              </div>
            </template>

            <template #item.valorPagoModerador="{ item, index }">
              <div class="text-center">
                <AppTextField clearable v-model="item.valorPagoModerador" :rules="[requiredValidator]"
                  @keyup="item.valorPagoModerador = formatoMoneda(item.valorPagoModerador)" />
              </div>
            </template>


            <template #item.vrServicio="{ item, index }">
              <div class="text-center">
                <AppTextField clearable v-model="item.vrServicio" :rules="[requiredValidator]"
                  @keyup="item.vrServicio = formatoMoneda(item.vrServicio)" />
              </div>
            </template>
            <template #item.conceptoRecaudo="{ item, index }">
              <div class="text-center">
                <SelectInfinite item-value="code" :key="index" v-model="item.conceptoRecaudo"
                  :select="selectsInfinites['conceptoRecaudo'][index]" clearable :rules="[requiredValidator]">
                </SelectInfinite>
              </div>
            </template>

            <template #item.actions="{ item, index }">
              <IconBtn density="compact">
                <VIcon icon="tabler-dots-vertical" />
                <VMenu activator="parent">
                  <VList>
                    <VListItem @click="deleteData(index)"><VIcon icon="tabler-trash" start/>Eliminar</VListItem>
                  </VList>
                </VMenu>
              </IconBtn>
            </template>
            <template #no-data> No se encontraron datos </template>
          </VDataTable>
        </VForm>

      </VCardText>

      <VCardText class="d-flex justify-end gap-3 flex-wrap mt-5">
        <VBtn variant="outlined" :disabled="loading.table" :loading="loading.table" color="primary" @click="goBack()">
          <VIcon start icon="tabler-arrow-left" />
          Regresar a usuarios
        </VBtn>
        <VBtn :disabled="loading.table" :loading="loading.table" @click="saveData()" color="primary">
          Guardar Consultas
          <VIcon end icon="tabler-device-floppy" />
        </VBtn>
      </VCardText>
    </VCard>
  </div>
</template>
