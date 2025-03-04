<script setup lang="ts">
import ApiReferenceLayout from '@/components/ApiReferenceLayout.vue'
import ClassicHeader from '@/components/ClassicHeader.vue'
import { DarkModeIconToggle } from '@/components/DarkModeToggle'
import { SearchButton } from '@/features/Search'
import type { ReferenceLayoutProps, ReferenceLayoutSlots } from '@/types'
import { computed } from 'vue'

const props = defineProps<ReferenceLayoutProps>()

defineEmits<{
  (e: 'toggleDarkMode'): void
  (e: 'updateContent', v: string): void
}>()

const slots = defineSlots<ReferenceLayoutSlots>()

// Override the sidebar value and hide it
const config = computed(() => ({ ...props.configuration, showSidebar: false }))
</script>
<template>
  <ApiReferenceLayout
    :configuration="config"
    :parsedSpec="parsedSpec"
    :rawSpec="rawSpec"
    @updateContent="$emit('updateContent', $event)">
    <!-- Expose all layout slots upwards -->
    <template
      v-for="(_, name) in slots"
      #[name]="slotProps">
      <slot
        :name="name"
        v-bind="slotProps || {}"></slot>
    </template>
    <template #content-start="{ spec }">
      <ClassicHeader>
        <SearchButton
          v-if="!props.configuration.hideSearch"
          class="t-doc__sidebar"
          :searchHotKey="config.searchHotKey"
          :spec="spec" />
        <template #dark-mode-toggle>
          <DarkModeIconToggle
            v-if="!props.configuration.hideDarkModeToggle"
            :isDarkMode="isDark"
            @toggleDarkMode="$emit('toggleDarkMode')" />
        </template>
      </ClassicHeader>
    </template>
  </ApiReferenceLayout>
</template>
