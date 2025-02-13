<template>
  <VButton
    id="filter-button"
    :variant="variant"
    size="disabled"
    class="align-center flex-shrink-0 gap-2 self-center py-2 font-semibold focus-visible:border-tx focus-visible:ring focus-visible:ring-pink"
    :class="
      filtersAreApplied
        ? 'flex-shrink-0 px-3'
        : 'h-10 w-10 px-0 md:h-auto md:w-auto md:px-3'
    "
    :pressed="pressed"
    :disabled="disabled"
    aria-controls="filters"
    :aria-label="mdMinLabel"
    @click="$emit('toggle')"
    @keydown.tab.exact="$emit('tab', $event)"
  >
    <VIcon
      :class="filtersAreApplied ? 'hidden' : 'block'"
      :icon-path="filterIcon"
    />
    <span class="hidden md:inline-block">{{ mdMinLabel }}</span>
    <span class="md:hidden" :class="!filtersAreApplied && 'hidden'">{{
      smMaxLabel
    }}</span>
  </VButton>
</template>

<script lang="ts">
import { computed, defineComponent, inject, ref } from '@nuxtjs/composition-api'

import { useSearchStore } from '~/stores/search'
import type { ButtonVariant } from '~/types/button'
import { defineEvent } from '~/types/emits'
import { useI18n } from '~/composables/use-i18n'

import VButton from '~/components/VButton.vue'
import VIcon from '~/components/VIcon/VIcon.vue'

import filterIcon from '~/assets/icons/filter.svg'

export default defineComponent({
  name: 'VFilterButtonOld',
  components: {
    VIcon,
    VButton,
  },
  props: {
    pressed: {
      type: Boolean,
      default: false,
    },
    disabled: {
      type: Boolean,
      default: false,
    },
  },
  emits: {
    tab: defineEvent<[KeyboardEvent]>(),
    toggle: defineEvent(),
  },
  setup() {
    const i18n = useI18n()
    const searchStore = useSearchStore()
    const isMinScreenMd = inject('isMinScreenMd', ref(false))
    const isHeaderScrolled = inject('isHeaderScrolled', ref(false))
    const filterCount = computed(() => searchStore.appliedFilterCount)
    const filtersAreApplied = computed(() => filterCount.value > 0)

    /**
     * Determine the visual style of the button
     * based on the viewport, the application of filters, and scrolling.
     */
    const variant = computed(() => {
      // Show the bordered state by default, unless below md
      let value: ButtonVariant = isMinScreenMd.value
        ? 'action-menu-bordered'
        : 'action-menu'

      if (isHeaderScrolled.value) {
        value = 'action-menu'
      }
      if (filtersAreApplied.value) {
        value = 'action-menu-muted'
      }
      return value
    })

    /**
     * This label's verbosity makes it useful for the aria-label
     * where it is also used, especially on mobile where the
     * label would just be the number of applied filters, and therefore
     * basically useless as far as a label is concerned!
     */
    const mdMinLabel = computed(() =>
      filtersAreApplied.value
        ? i18n.tc('header.filter-button.with-count', filterCount.value)
        : i18n.t('header.filter-button.simple')
    )

    const smMaxLabel = computed(() =>
      isHeaderScrolled.value
        ? filterCount.value
        : i18n.tc('header.filter-button.with-count', filterCount.value)
    )

    return {
      filterCount,
      filterIcon,
      mdMinLabel,
      smMaxLabel,
      variant,
      isHeaderScrolled,
      filtersAreApplied,
    }
  },
})
</script>
