<script setup lang="ts">

import ModalMassUpload from "@/components/CustomComponents/File/ModalMassUpload.vue";


//ModalMassUpload
const refModalMassUpload = ref()

const openModalMassUpload = () => {
  refModalMassUpload.value.openModal("0196911c-3982-7158-a916-a37be3d3762a", "User");
}



//ModalShowFiles
const refModalShowFiles = ref()

const openModalShowFiles = () => {
  refModalShowFiles.value.openModal("0196911c-3982-7158-a916-a37be3d3762a", "User")
}

</script>

<template>
  <div>

    <VBtn color="success" @click="openModalMassUpload()"> Subirsoportes</VBtn>


    <VBtn class="me-2 mb-2" @click="openModalShowFiles">
      <template #prepend>
        <VIcon start icon="tabler-files" />
      </template>
      Soportes
    </VBtn>

    <ModalMassUpload ref="refModalMassUpload" />

    <ModalShowFiles ref="refModalShowFiles"></ModalShowFiles>



  </div>
</template>
