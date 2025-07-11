<script setup lang="ts">
import FileUploadModal from './FileUploadModal.vue'

const showUploadModal = ref(false)

const openUploadModal = () => {
  showUploadModal.value = true
}
</script>

<template>
  <div>
    <VBtn prepend-icon="tabler-upload" @click="openUploadModal" elevation="2">
      Start Upload
    </VBtn>

    <FileUploadModal v-model="showUploadModal" :maxFileSizeMB="30" :allowed-extensions="['.jpg', '.png', '.pdf']"
      :max-files="5" />
  </div>
</template>
