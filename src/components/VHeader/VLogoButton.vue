<template>
  <VButton
    as="VLink"
    href="/"
    :aria-label="$t('header.home-link')"
    variant="plain"
    size="disabled"
    class="max-w-min text-dark-charcoal hover:bg-yellow"
  >
    <VLogoLoader
      :status="isFetching ? 'loading' : 'idle'"
      :auto-resize="autoResizeLogo"
    />
  </VButton>
</template>
<script>
import { defineComponent } from '@nuxtjs/composition-api'

import VLogoLoader from '~/components/VLogoLoader/VLogoLoader.vue'
import VButton from '~/components/VButton.vue'

export default defineComponent({
  name: 'VLogoButton',
  components: { VLogoLoader, VButton },
  props: {
    isFetching: {
      type: Boolean,
      default: false,
    },
    isSearchRoute: {
      type: Boolean,
      default: false,
    },
    autoResizeLogo: {
      type: Boolean,
      default: true,
    },
  },
})
</script>
