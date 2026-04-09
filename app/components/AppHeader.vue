<script setup lang="ts">
import type { NavigationMenuItem } from '@nuxt/ui'

defineProps<{
  links: NavigationMenuItem[]
}>()
</script>

<template>
  <div class="fixed top-2 sm:top-4 mx-auto left-1/2 transform -translate-x-1/2 z-10">
    <!-- Gamer HUD corner accents -->
    <div class="relative">
      <!-- Top-left corner -->
      <span
        class="absolute -top-1 -left-1 w-3 h-3 pointer-events-none"
        style="border-top: 1.5px solid #00ffff; border-left: 1.5px solid #00ffff; opacity: 0.7;"
      />
      <!-- Top-right corner -->
      <span
        class="absolute -top-1 -right-1 w-3 h-3 pointer-events-none"
        style="border-top: 1.5px solid #00ffff; border-right: 1.5px solid #00ffff; opacity: 0.7;"
      />
      <!-- Bottom-left corner -->
      <span
        class="absolute -bottom-1 -left-1 w-3 h-3 pointer-events-none"
        style="border-bottom: 1.5px solid #7b2fff; border-left: 1.5px solid #7b2fff; opacity: 0.7;"
      />
      <!-- Bottom-right corner -->
      <span
        class="absolute -bottom-1 -right-1 w-3 h-3 pointer-events-none"
        style="border-bottom: 1.5px solid #7b2fff; border-right: 1.5px solid #7b2fff; opacity: 0.7;"
      />

      <UNavigationMenu
        :items="links"
        variant="link"
        color="neutral"
        class="px-2 sm:px-4 border rounded-full shadow-lg"
        style="
          background: rgba(6,6,20,0.75);
          backdrop-filter: blur(18px);
          border-color: rgba(0,255,255,0.18);
          box-shadow: 0 0 18px rgba(0,255,255,0.08), 0 0 40px rgba(123,47,255,0.06), inset 0 0 20px rgba(0,255,255,0.03);
        "
        :ui="{
          link: 'px-2 py-1',
          linkLeadingIcon: 'hidden'
        }"
      >
        <template #list-trailing>
          <ColorModeButton />
        </template>
      </UNavigationMenu>
    </div>
  </div>
</template>
