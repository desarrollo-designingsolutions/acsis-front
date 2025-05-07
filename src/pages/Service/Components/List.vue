<script setup lang="ts">
import { useAuthenticationStore } from "@/stores/useAuthenticationStore";

const { toast } = useToast();

const props = defineProps<{
  invoice_id: string
}>()

const invoice_id = props.invoice_id;
const authenticationStore = useAuthenticationStore();
const servicesIds = ref<Array<string>>([]);

const currentTab = ref(0)
const servicesCount = ref({
  consultas: 0,
  procedimientos: 0,
  urgencias: 0,
  hospitalizacion: 0,
  recienNacidos: 0,
  medicamentos: 0,
  otrosServicios: 0
})
const keyComponent = ref(0)
const dataUser = ref({
  servicios: {
    consultas: [],
    procedimientos: [],
    urgencias: [],
    hospitalizacion: [],
    recienNacidos: [],
    medicamentos: [],
    otrosServicios: []
  }
}) 
</script>

<template>
  <div>
    <VCard class="mt-5" title="Listado de servicios">
      <VCardText>

        <VTabs v-model="currentTab" grow>
          <VTab>
            <span>Consultas</span>
            <VBadge :content="servicesCount.consultas" :offset-x="-18" :offset-y="0" />
          </VTab>
          <VTab>
            <span>Procedimientos</span>
            <VBadge :content="servicesCount.procedimientos" :offset-x="-18" :offset-y="0" />
          </VTab>
          <VTab>
            <span>Urgencias</span>
            <VBadge :content="servicesCount.urgencias" :offset-x="-18" :offset-y="0" />
          </VTab>
          <VTab>
            <span>Hospitalización</span>
            <VBadge :content="servicesCount.hospitalizacion" :offset-x="-18" :offset-y="0" />
          </VTab>
          <VTab>
            <span>Recien nacidos</span>
            <VBadge :content="servicesCount.recienNacidos" :offset-x="-18" :offset-y="0" />
          </VTab>
          <VTab>
            <span>Medicamentos</span>
            <VBadge :content="servicesCount.medicamentos" :offset-x="-18" :offset-y="0" />
          </VTab>
          <VTab>
            <span>Otros servicios</span>
            <VBadge :content="servicesCount.otrosServicios" :offset-x="-18" :offset-y="0" />
          </VTab>
        </VTabs>

        <VWindow v-model="currentTab" class="my-5">
          <VWindowItem>
            <QueriesForm :key="keyComponent" :data-list="dataUser?.servicios?.consultas"></QueriesForm>
          </VWindowItem>
          <VWindowItem>
            ProcedureForm
            <!-- <ProcedureForm :key="keyComponent" :data-list="dataUser?.servicios?.procedimientos"></ProcedureForm> -->
          </VWindowItem>
          <VWindowItem>
            EmergenciesForm
            <!-- <EmergenciesForm :key="keyComponent" :data-list="dataUser?.servicios?.urgencias"></EmergenciesForm> -->
          </VWindowItem>
          <VWindowItem>
            HospitalizationForm
            <!-- <HospitalizationForm :key="keyComponent" :data-list="dataUser?.servicios?.hospitalizacion">
            </HospitalizationForm> -->
          </VWindowItem>
          <VWindowItem>
            NewlyBornForm
            <!-- <NewlyBornForm :key="keyComponent" :data-list="dataUser?.servicios?.recienNacidos"></NewlyBornForm> -->
          </VWindowItem>
          <VWindowItem>
            MedicinesForm
            <!-- <MedicinesForm :key="keyComponent" :data-list="dataUser?.servicios?.medicamentos"></MedicinesForm> -->
          </VWindowItem>
          <VWindowItem>
            OtherServicesForm
            <!-- <OtherServicesForm :key="keyComponent" :data-list="dataUser?.servicios?.otrosServicios"></OtherServicesForm> -->
          </VWindowItem>
        </VWindow>
      </VCardText>
    </VCard>
  </div>
</template>
