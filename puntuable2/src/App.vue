<script setup lang="ts">
import { TresCanvas } from '@tresjs/core'

import mundo from './components/mundo.vue'
</script>

<template>
  <TresCanvas
    clear-color="#0f0f0f"
    window-size>
    <mundo />
    <barra-lateral />
  </TresCanvas>
</template>