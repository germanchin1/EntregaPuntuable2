<script setup lang="ts">
import type { TresObject } from '@tresjs/core'
import { useLoop } from '@tresjs/core'
import { shallowRef } from 'vue'
import { OrbitControls } from '@tresjs/cientos'

const { onBeforeRender } = useLoop()

const boxRef = shallowRef<TresObject | null>(null)

onBeforeRender(({ elapsed }) => {
  if (boxRef.value) {
    boxRef.value.rotation.y = elapsed
    boxRef.value.rotation.z = elapsed
  }
})
</script>

<template>
  <TresPerspectiveCamera make-default :position="[5, 5, 5]" :look-at="[0, 0, 0]" />
  <OrbitControls :target="[0, 0, 0]" :enable-damping="true" />
  <TresAmbientLight :intensity="0.5" color="white" />
  <TresDirectionalLight :position="[0, 2, 4]" :intensity="1" cast-shadow />
  <TresGridHelper :args="[15, 15]" />
</template>