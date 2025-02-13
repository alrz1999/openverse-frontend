<template>
  <div
    ref="nodeRef"
    class="flex items-stretch justify-end text-sr md:text-base"
  >
    <VFilterButtonOld
      ref="buttonRef"
      class="self-stretch"
      :class="visibleRef ? 'hidden md:flex' : 'flex'"
      :pressed="visibleRef"
      :disabled="disabled"
      aria-haspopup="dialog"
      :aria-expanded="visibleRef"
      @toggle="onTriggerClick"
      @tab="onTab"
    />
    <template v-if="visibleRef">
      <VTeleport v-if="isMinScreenMd" to="sidebar">
        <VSearchGridFilter @close="onTriggerClick" />
      </VTeleport>
      <VModalContent
        v-else
        :hide="close"
        :visible="true"
        :trigger-element="triggerElement"
        :aria-label="$t('header.filter-button.simple')"
      >
        <VSearchGridFilter @close="onTriggerClick" />
      </VModalContent>
    </template>
  </div>
</template>

<script lang="ts">
import {
  defineComponent,
  ref,
  watch,
  computed,
  onMounted,
  inject,
  Ref,
  toRef,
  onBeforeUnmount,
} from '@nuxtjs/composition-api'

import { Portal as VTeleport } from 'portal-vue'

import { useBodyScrollLock } from '~/composables/use-body-scroll-lock'
import { useFilterSidebarVisibility } from '~/composables/use-filter-sidebar-visibility'
import { useFocusFilters } from '~/composables/use-focus-filters'

import { Focus } from '~/utils/focus-management'
import { defineEvent } from '~/types/emits'
import local from '~/utils/local'
import { env } from '~/utils/env'
import { useSearchStore } from '~/stores/search'

import VModalContent from '~/components/VModal/VModalContent.vue'
import VSearchGridFilter from '~/components/VFilters/VSearchGridFilter.vue'
import VFilterButtonOld from '~/components/VHeaderOld/VFilterButtonOld.vue'

export default defineComponent({
  name: 'VHeaderFilter',
  components: {
    VFilterButtonOld,
    VModalContent,
    VSearchGridFilter,
    VTeleport,
  },
  props: {
    disabled: {
      type: Boolean,
      default: false,
    },
  },
  emits: {
    /**
     * Fires when the popover opens, regardless of reason. There are no extra parameters.
     */
    open: defineEvent(),
    /**
     * Fires when the popover closes, regardless of reason. There are no extra parameters.
     */
    close: defineEvent(),
  },
  setup(props, { emit }) {
    const nodeRef = ref<HTMLElement | null>(null)
    const visibleRef = ref(false)
    const filterSidebar = useFilterSidebarVisibility()
    const disabledRef = toRef(props, 'disabled')

    const isMinScreenMd: Ref<boolean> = inject('isMinScreenMd')

    const open = () => (visibleRef.value = true)
    const close = () => (visibleRef.value = false)
    const { lock, unlock } = useBodyScrollLock({ nodeRef })

    onMounted(() => {
      // We default to show the filter on desktop, and only close it if the user has
      // explicitly closed it before.
      const localFilterState = !(
        local.getItem(env.filterStorageKey) === 'false'
      )
      const searchStore = useSearchStore()
      if (!isMinScreenMd.value) {
        local.setItem(env.filterStorageKey, 'false')
      } else {
        const visible = searchStore.searchTypeIsSupported && localFilterState
        filterSidebar.setVisibility(visible)
        if (visible) {
          open()
        }
      }
    })

    onBeforeUnmount(() => unlock())

    watch(visibleRef, (visible) => {
      filterSidebar.setVisibility(visible)
      visible ? emit('open') : emit('close')
      if (!isMinScreenMd.value) {
        visible ? lock() : unlock()
      }
    })

    watch(disabledRef, (disabled) => {
      if (disabled && visibleRef.value) {
        close()
      }
    })

    // Lock the scroll when the screen changes to below Md if filters are open.
    watch(isMinScreenMd, (isMd) => {
      if (!isMd && visibleRef.value) {
        lock()
      }
    })

    const onTriggerClick = () => {
      visibleRef.value = !visibleRef.value
    }

    const focusFilters = useFocusFilters()
    /**
     * Focus the first element in the sidebar when navigating from the VFilterButton
     * using keyboard `Tab` key.
     */
    const onTab = (event: KeyboardEvent) => {
      focusFilters.focusFilterSidebar(event, Focus.First)
    }

    const triggerElement = computed(() =>
      nodeRef.value?.firstChild
        ? (nodeRef.value?.firstChild as HTMLElement)
        : null
    )

    return {
      nodeRef,
      visibleRef,
      triggerElement,

      open,
      close,
      onTriggerClick,
      onTab,
      isMinScreenMd,
    }
  },
})
</script>
