<script setup lang="ts">
import { TresCanvas } from '@tresjs/core'

import mundo from './components/mundo.vue'
import barraLateral from './components/barralateral.vue'
</script>

<template>
  <TresCanvas
    clear-color="#2a2b2e"
    window-size
  >
    <mundo />
    <barra-lateral />
  </TresCanvas>
</template>