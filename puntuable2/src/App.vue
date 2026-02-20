<script setup lang="ts">
import { TresCanvas } from '@tresjs/core'

import mundo from './components/mundo.vue'
import barraLateral from './components/barralateral.vue'
</script>

<template>
  <barra-lateral />
  <TresCanvas
    clear-color="#0f0f0f"
    window-size>
    <mundo />
  </TresCanvas>
</template>