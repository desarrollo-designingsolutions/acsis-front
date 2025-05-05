<script setup lang="ts">

import ModalMassUpload from "@/components/CustomComponents/File/ModalMassUpload.vue";

const id = "019686bb-ee3d-7125-8edb-67fa2eb14600";
const type = "User";

//ModalMassUpload
const refModalMassUpload = ref()

const openModalMassUpload = () => {
  refModalMassUpload.value.openModal(id, type);
}



//ModalShowFiles
const refModalShowFiles = ref()

const openModalShowFiles = () => {
  refModalShowFiles.value.openModal(id, type)
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
