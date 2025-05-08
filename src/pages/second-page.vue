<script setup lang="ts">
import ListService from "@/pages/Service/Components/List.vue";


const invoice_id = ref('0196ac8c-07ea-7248-9e62-6956c264c9e9')
</script>

<template>
  <div>

    <ListService :invoice_id="invoice_id" />

  </div>
</template>
