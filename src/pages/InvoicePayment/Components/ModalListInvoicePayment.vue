<script setup lang="ts">
import ListInvoicePayment from "@/pages/InvoicePayment/Components/List.vue";
import { useAuthenticationStore } from "@/stores/useAuthenticationStore";

const authenticationStore = useAuthenticationStore();

const emit = defineEmits(["execute"])

const titleModal = ref<string>("Listado de pagos")
const isDialogVisible = ref<boolean>(false)
const disabledFiledsView = ref<boolean>(false)
const showList = ref<boolean>(false)

const isLoading = ref<boolean>(false)

const form = ref({
  invoice_id: null as string | null,
})

const handleClearForm = () => {
  for (const key in form.value) {
    form.value[key] = null;
  }
}

const handleDialogVisible = () => {
  isDialogVisible.value = !isDialogVisible.value;
  showList.value = !showList.value;
  handleClearForm()
};

const openModal = async ({ invoice_id }: any, disabled: boolean = false) => {
  disabledFiledsView.value = disabled

  handleDialogVisible();

  form.value.invoice_id = invoice_id;
  titleModal.value = `Listado de pagos`
};

defineExpose({
  openModal
})

</script>

<template>
  <div>
    <VDialog v-model="isDialogVisible" max-width="70rem" transition="dialog-transition" persistent>
      <DialogCloseBtn @click="handleDialogVisible" />
      <VCard :loading="isLoading" :disabled="isLoading" class="w-100">
        <!-- Toolbar -->

        <VToolbar color="primary" class="rounded-t">
          <VToolbarTitle class="text-h6 font-weight-medium">
            {{ titleModal }}
          </VToolbarTitle>
        </VToolbar>

        <VCardText class="px-0">
          <ListInvoicePayment v-if="showList" :service_id="form.id"></ListInvoicePayment>
        </VCardText>
      </VCard>
    </VDialog>

  </div>
</template>
