<template>
  <VPopover
    ref="contentMenuPopover"
    class="flex items-stretch"
    :label="$t('search-type.label').toString()"
    placement="bottom-end"
    :clippable="true"
  >
    <template #trigger="{ a11yProps }">
      <VSearchTypeButton
        :a11y-props="a11yProps"
        aria-controls="content-switcher-popover"
      />
    </template>
    <VSearchTypes
      id="content-switcher-popover"
      size="medium"
      :use-links="true"
      @select="closePopover"
    />
  </VPopover>
</template>

<script lang="ts">
import { defineComponent, ref } from '@nuxtjs/composition-api'

import VPopover from '~/components/VPopover/VPopover.vue'
import VSearchTypeButton from '~/components/VContentSwitcher/VSearchTypeButton.vue'
import VSearchTypes from '~/components/VContentSwitcher/VSearchTypes.vue'

import checkIcon from '~/assets/icons/checkmark.svg'

export default defineComponent({
  name: 'VSearchTypePopover',
  components: {
    VPopover,
    VSearchTypeButton,
    VSearchTypes,
  },
  setup() {
    const contentMenuPopover = ref<InstanceType<typeof VPopover> | null>(null)

    const closePopover = () => {
      contentMenuPopover.value?.close()
    }

    return {
      checkIcon,
      closePopover,
      contentMenuPopover,
    }
  },
})
</script>
