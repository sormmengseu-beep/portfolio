<script setup lang="ts">
const now = ref('')
onMounted(() => {
  const update = () => {
    const d = new Date()
    now.value = d.toLocaleTimeString('en-GB', { hour: '2-digit', minute: '2-digit', second: '2-digit' })
  }
  update()
  setInterval(update, 1000)
})
</script>

<template>
  <div class="min-h-screen" style="background-color:#06060f;">
    <!-- Radial vignette / ambient glow -->
    <div
      class="pointer-events-none fixed inset-0 z-1"
      style="background: radial-gradient(ellipse at 50% 0%, rgba(123,47,255,0.14) 0%, transparent 65%), radial-gradient(ellipse at 50% 100%, rgba(0,255,136,0.06) 0%, transparent 60%);"
    />

    <!-- HUD: top-left corner status -->
    <div
      class="pointer-events-none fixed top-3 left-3 z-20 flex flex-col gap-1"
      style="font-family: 'Courier New', monospace; font-size: 0.6rem; letter-spacing: 0.1em; color: rgba(0,255,255,0.45);"
    >
      <span>SYS :: ONLINE</span>
      <span>BUILD v2.0.0</span>
      <ClientOnly>
        <span>{{ now }}</span>
      </ClientOnly>
    </div>

    <!-- HUD: top-right corner -->
    <div
      class="pointer-events-none fixed top-3 right-3 z-20 flex flex-col items-end gap-1"
      style="font-family: 'Courier New', monospace; font-size: 0.6rem; letter-spacing: 0.1em; color: rgba(123,47,255,0.5);"
    >
      <span>RENDER :: THREE.JS</span>
      <span>MODE :: CYBERPUNK</span>
    </div>

    <!-- HUD: bottom status bar -->
    <div
      class="pointer-events-none fixed bottom-0 left-0 right-0 z-20 flex items-center justify-between px-4 py-1"
      style="
        background: rgba(0,0,0,0.4);
        border-top: 1px solid rgba(0,255,255,0.08);
        font-family: 'Courier New', monospace;
        font-size: 0.6rem;
        letter-spacing: 0.12em;
      "
    >
      <span style="color: rgba(0,255,255,0.35);">◈ PORTFOLIO SYSTEM ACTIVE</span>

      <!-- Neon bar indicators -->
      <div class="flex items-center gap-2">
        <div class="flex gap-0.5">
          <span
            v-for="n in 8"
            :key="n"
            class="inline-block w-2 h-1.5 rounded-sm"
            :style="`background: ${n <= 6 ? 'rgba(0,255,255,0.5)' : 'rgba(0,255,255,0.1)'};`"
          />
        </div>
        <span style="color: rgba(0,255,255,0.35);">75% PWR</span>
      </div>

      <span style="color: rgba(123,47,255,0.45);">◈ FPS :: ∞</span>
    </div>

    <!-- Page screen-corner bracket decorations -->
    <!-- top-left -->
    <div class="pointer-events-none fixed top-0 left-0 z-20 w-8 h-8" style="border-top: 1.5px solid rgba(0,255,255,0.3); border-left: 1.5px solid rgba(0,255,255,0.3);" />
    <!-- top-right -->
    <div class="pointer-events-none fixed top-0 right-0 z-20 w-8 h-8" style="border-top: 1.5px solid rgba(0,255,255,0.3); border-right: 1.5px solid rgba(0,255,255,0.3);" />
    <!-- bottom-left -->
    <div class="pointer-events-none fixed bottom-0 left-0 z-20 w-8 h-8" style="border-bottom: 1.5px solid rgba(123,47,255,0.3); border-left: 1.5px solid rgba(123,47,255,0.3);" />
    <!-- bottom-right -->
    <div class="pointer-events-none fixed bottom-0 right-0 z-20 w-8 h-8" style="border-bottom: 1.5px solid rgba(123,47,255,0.3); border-right: 1.5px solid rgba(123,47,255,0.3);" />

    <!-- Content -->
    <div class="relative z-10">
      <UContainer
        class="sm:border-x border-white/5 pt-10"
        style="background: rgba(6,6,20,0.5); backdrop-filter: blur(10px);"
      >
        <AppHeader :links="navLinks" />
        <slot />
        <AppFooter />
      </UContainer>
    </div>
  </div>
</template>
