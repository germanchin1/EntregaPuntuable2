<script setup lang="ts">
import { OrbitControls } from '@tresjs/cientos'
import { onMounted, onUnmounted } from 'vue'
import { useTres } from '@tresjs/core'
import * as THREE from 'three'
import { DragControls } from 'three-stdlib'

const { scene, camera, renderer, controls } = useTres()

const draggable: any[] = []
let dragControls: DragControls;
let ultimoObjetoArrastrado: THREE.Object3D;

onMounted(() => {
  const activeCamera = camera.value
  //NI IDEA DE QUE HACE ESTO PERO ES PARA PODER ARRASTRAR LOS CUBOS
  const domElement = 'domElement' in renderer ? renderer.domElement : null
  if (!activeCamera || !domElement) {
    return
  }
const light = new THREE.AmbientLight(0xffffff, 1); // Luz blanca, intensidad 1
scene.value.add(light)


  const geometry = new THREE.BoxGeometry(1, 1, 3)
  const material = new THREE.MeshStandardMaterial({ color: 0xfcba03 })
  const material2 = new THREE.MeshStandardMaterial({ color: 0xacacac, side: THREE.BackSide /*ver solo por dentro */ })
  
  //CREAR LA HABITACIÓN
  const habitacion = new THREE.Mesh(
  new THREE.BoxGeometry(7, 5, 10), material2)
  habitacion.position.y = 2.5
  scene.value.add(habitacion)

  const cube = new THREE.Mesh(geometry, material)
  cube.position.y = 0.5
  draggable.push(cube)
  scene.value.add(cube)

  const cube2 = new THREE.Mesh(geometry, material)
  cube2.position.set(2, 0.5, 0)
  draggable.push(cube2)
  scene.value.add(cube2)

  const cube3 = new THREE.Mesh(geometry, material)
  cube3.position.set(-2, 0.5, 0)
  draggable.push(cube3)
  cube3.name
  scene.value.add(cube3)


  //NI IDEA DE QUE HACE ESTO PERO ES PARA PODER ARRASTRAR LOS CUBOS
  dragControls = new DragControls(draggable, activeCamera, domElement)


  //EMPEZAR A ESCUCHAR LOS EVENTOS DE DRAG
  dragControls.addEventListener('dragstart', (event) => {
    ultimoObjetoArrastrado = event.object
    console.log('dragstart', event.object)
    console.log("el ultimo objeto arrastrado es: ", ultimoObjetoArrastrado)
    if (controls.value) {
      controls.value.enabled = false
    }

    //ROTACIÓN
    window.addEventListener('keydown', (r) => {
      if (r.key === 'r' && ultimoObjetoArrastrado) {
        ultimoObjetoArrastrado.rotation.y += Math.PI / 8
        console.log(ultimoObjetoArrastrado.rotation.y)
      }
    })
  })

  dragControls.addEventListener('drag', () => {
    if (controls.value) {
      controls.value.enabled = false
    }
  if (cube.position.y !== 0.5 || cube2.position.y !== 0.5 || cube3.position.y !== 0.5) {
    cube.position.y = 0.5
    cube2.position.y = 0.5
    cube3.position.y = 0.5
  }

  })

  dragControls.addEventListener('dragend', (event) => {
    console.log('dragend', event.object)
    if (controls.value) {
      controls.value.enabled = true
    }
  })
})


</script>

<template>
  <TresPerspectiveCamera make-default :position="[5, 5, 5]" :look-at="[0, 0, 0]" />
  <OrbitControls make-default :target="[0, 0, 0]" :enable-damping="true" />
  <TresAmbientLight :intensity="0.5" color="#4287f5" />
  <TresDirectionalLight :position="[0, 2, 4]" :intensity="0.4" cast-shadow />
  <TresGridHelper :args="[15, 15]" />
</template>