<script lang="ts" setup>
defineOptions({
  name: 'AppSelect',
  inheritAttrs: false,
})

const elementId = computed(() => {
  const attrs = useAttrs()
  const _elementIdToken = attrs.id || attrs.label

  return _elementIdToken ? `app-select-${_elementIdToken}-${Math.random().toString(36).slice(2, 7)}` : undefined
})

const label = computed(() => useAttrs().label as string | undefined)
const requiredField = computed(
  () => useAttrs().requiredField as boolean | undefined
);
const tooltip = computed(
  () => useAttrs().tooltip as object | undefined
);
</script>

<template>
  <div class="app-select flex-grow-1" :class="$attrs.class">
    <VLabel v-if="label" :for="elementId" class="mb-1 text-body-2 text-high-emphasis d-block text-wrap">
      {{ label }}
      <span v-if="requiredField">&nbsp; <b class="text-warning">*</b></span>
      <VTooltip v-if="tooltip" :location="tooltip.location ?? 'top'">
        <template #activator="{ props }">
          <VIcon v-if="tooltip.icon" v-bind="props" :icon="tooltip.icon" />
        </template>
        <span>{{ tooltip.text }}</span>
      </VTooltip>
    </VLabel>


    <VSelect v-bind="{
      ...$attrs,
      class: null,
      label: undefined,
      variant: 'outlined',
      id: elementId,
      menuProps: { contentClass: ['app-inner-list', 'app-select__content', 'v-select__content', $attrs.multiple !== undefined ? 'v-list-select-multiple' : ''] },
    }">
      <template v-for="(_, name) in $slots" #[name]="slotProps">
        <slot :name="name" v-bind="slotProps || {}" />
      </template>
    </VSelect>
  </div>
</template>
