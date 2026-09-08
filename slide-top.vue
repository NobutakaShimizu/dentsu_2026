<script setup lang="ts">
import { computed } from 'vue'
import { configs, useSlideContext } from '@slidev/client'

const { $page, $frontmatter } = useSlideContext()

const slug = computed(() => (configs as { neversink_slug?: string }).neversink_slug || 'dentsu_2026')
const isCover = computed(() => ($frontmatter.layout || ($page.value === 1 ? 'cover' : 'default')) === 'cover')
const qrUrl = computed(() => `https://nobutakashimizu.github.io/${slug.value}/${$page.value}`)
</script>

<template>
  <div v-if="!isCover" class="qr-code-fixed" aria-hidden="true">
    <QRCode :value="qrUrl" :size="80" render-as="svg" />
  </div>
</template>
