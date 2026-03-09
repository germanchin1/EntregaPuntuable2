<script setup lang="ts">
import { OrbitControls } from '@tresjs/cientos'
import { onMounted, onUnmounted } from 'vue'
import { useTres } from '@tresjs/core'
import * as THREE from 'three'
import { DragControls, MTLLoader, OBJLoader } from 'three-stdlib'

const { scene, camera, renderer, controls } = useTres()

type FurnitureType = 'silla' | 'mesa' | 'cama'

const draggable: THREE.Object3D[] = []
const modelCache: Record<FurnitureType, THREE.Group | undefined> = {
  silla: undefined,
  mesa: undefined,
  cama: undefined,
}

let dragControls: DragControls | null = null
let ultimoObjetoArrastrado: THREE.Object3D | null = null

const modelMap: Record<FurnitureType, { objUrl: string; mtlUrl: string; scaleTarget: number; rotateX?: number }> = {
  silla: {
    objUrl: new URL('../assets/3d/chair/chair.obj', import.meta.url).href,
    mtlUrl: new URL('../assets/3d/chair/chair.mtl', import.meta.url).href,
    scaleTarget: 1.4,
  },
  cama: {
    objUrl: new URL('../assets/3d/bed/bed.obj', import.meta.url).href,
    mtlUrl: new URL('../assets/3d/bed/bed.mtl', import.meta.url).href,
    scaleTarget: 2.2,
    rotateX: -Math.PI / 2,
  },
  mesa: {
    objUrl: new URL('../assets/3d/table/table.obj', import.meta.url).href,
    mtlUrl: new URL('../assets/3d/table/table.mtl', import.meta.url).href,
    scaleTarget: 1.4,
  },
}

onMounted(() => {
  const activeCamera = camera.value

  const domElement = 'domElement' in renderer ? renderer.domElement : null
  if (!activeCamera || !domElement) {
    return
  }

  const light = new THREE.AmbientLight(0xffffff, 1)
  scene.value.add(light)

  const material2 = new THREE.MeshStandardMaterial({ color: 0xacacac, side: THREE.BackSide /*ver solo por dentro */ })
  
  const habitacion = new THREE.Mesh(new THREE.BoxGeometry(7, 5, 10), material2)
  habitacion.position.y = 2.5
  scene.value.add(habitacion)

  const setOrbitEnabled = (enabled: boolean) => {
    if (controls.value) {
      controls.value.enabled = enabled
    }
  }

  const getDraggableRoot = (object: THREE.Object3D | null): THREE.Object3D | null => {
    let current: THREE.Object3D | null = object
    while (current) {
      if (draggable.includes(current)) {
        return current
      }
      current = current.parent
    }
    return null
  }

  const configurarDragControls = () => {
    if (!dragControls) return

    dragControls.addEventListener('dragstart', (event: any) => {
      const root = getDraggableRoot(event.object)
      if (root) {
        ultimoObjetoArrastrado = root
        event.object = root
      } else {
        ultimoObjetoArrastrado = event.object ?? null
      }
      setOrbitEnabled(false)
    })

    dragControls.addEventListener('drag', (event: any) => {
      setOrbitEnabled(false)
      const root = getDraggableRoot(event.object)
      if (root) {
        ultimoObjetoArrastrado = root
      }
      if (ultimoObjetoArrastrado && ultimoObjetoArrastrado.position.y !== 0.5) {
        ultimoObjetoArrastrado.position.y = 0.5
      }
    })

    dragControls.addEventListener('dragend', () => {
      setOrbitEnabled(true)
    })
  }

  const recrearDragControls = () => {
    if (dragControls) dragControls.dispose()
    dragControls = new DragControls(draggable, activeCamera, domElement)
    ;(dragControls as any).recursive = false
    configurarDragControls()
  }

  const normalizarModelo = (obj: THREE.Group, tipo: FurnitureType): THREE.Object3D => {
    const config = modelMap[tipo]
    const model = obj.clone(true)
    const grupoVisual = new THREE.Group()
    grupoVisual.add(model)

    if (config.rotateX) {
      grupoVisual.rotation.x = config.rotateX
    }

    const box = new THREE.Box3().setFromObject(grupoVisual)
    const size = new THREE.Vector3()
    box.getSize(size)

    const maxAxis = Math.max(size.x, size.y, size.z) || 1
    const escala = config.scaleTarget / maxAxis
    grupoVisual.scale.setScalar(escala)

    grupoVisual.updateMatrixWorld(true)
    const boxEscalado = new THREE.Box3().setFromObject(grupoVisual)
    const centerEscalado = new THREE.Vector3()
    boxEscalado.getCenter(centerEscalado)

    grupoVisual.position.set(-centerEscalado.x, -boxEscalado.min.y, -centerEscalado.z)
    grupoVisual.updateMatrixWorld(true)

    const boxFinal = new THREE.Box3().setFromObject(grupoVisual)
    const sizeFinal = new THREE.Vector3()
    boxFinal.getSize(sizeFinal)

    grupoVisual.traverse((child) => {
      if ((child as THREE.Mesh).isMesh) {
        child.raycast = () => null
      }
    })

    const dragGeometry = new THREE.BoxGeometry(
      Math.max(sizeFinal.x, 0.4),
      Math.max(sizeFinal.y, 0.4),
      Math.max(sizeFinal.z, 0.4),
    )
    const dragMaterial = new THREE.MeshBasicMaterial({
      transparent: true,
      opacity: 0,
      depthWrite: false,
    })
    const dragHandle = new THREE.Mesh(dragGeometry, dragMaterial)
    dragHandle.position.set(0, 0.5, 0)
    dragHandle.name = tipo
    dragHandle.add(grupoVisual)

    return dragHandle
  }

  const loadRawModel = async (tipo: FurnitureType): Promise<THREE.Group> => {
    const config = modelMap[tipo]
    if (modelCache[tipo]) return modelCache[tipo]

    const mtlLoader = new MTLLoader()
    const materials = await mtlLoader.loadAsync(config.mtlUrl)
    materials.preload()

    const objLoader = new OBJLoader()
    objLoader.setMaterials(materials)
    const obj = await objLoader.loadAsync(config.objUrl) as THREE.Group
    modelCache[tipo] = obj
    return obj
  }

  const createFallback = (tipo: FurnitureType): THREE.Mesh => {
    const geometry = new THREE.BoxGeometry(1, 1, tipo === 'cama' ? 2 : 1)
    const colors: Record<FurnitureType, number> = {
      silla: 0xff0000,
      mesa: 0x00ff00,
      cama: 0x0000ff,
    }
    const material = new THREE.MeshStandardMaterial({ color: colors[tipo] })
    const mesh = new THREE.Mesh(geometry, material)
    mesh.position.set(0, 0.5, 0)
    mesh.name = tipo
    return mesh
  }

  const addDraggableObject = (object: THREE.Object3D) => {
    draggable.push(object)
    scene.value.add(object)
    recrearDragControls()
  }

  const crearMueble = async (tipo: FurnitureType) => {
    try {
      const rawModel = await loadRawModel(tipo)
      addDraggableObject(normalizarModelo(rawModel, tipo))
    } catch (error) {
      console.error(`Error cargando el modelo ${tipo}:`, error)
      addDraggableObject(createFallback(tipo))
    }
  }

  const onDragOver = (e: DragEvent) => {
    e.preventDefault()
  }

  const onDrop = (e: DragEvent) => {
    e.preventDefault()
    e.stopPropagation()
    const tipo = e.dataTransfer?.getData('tipo-mueble') as FurnitureType | ''
    if (tipo && tipo in modelMap) {
      void crearMueble(tipo)
    }
  }

  const onKeydown = (tecla: KeyboardEvent) => {
    if (tecla.key.toLowerCase() === 'r' && ultimoObjetoArrastrado) {
      ultimoObjetoArrastrado.rotation.y += Math.PI / 4
    }
  }

  domElement.addEventListener('dragover', onDragOver)
  domElement.addEventListener('drop', onDrop)
  document.addEventListener('keydown', onKeydown)

  recrearDragControls()

  onUnmounted(() => {
    domElement.removeEventListener('dragover', onDragOver)
    domElement.removeEventListener('drop', onDrop)
    document.removeEventListener('keydown', onKeydown)
    if (dragControls) {
      dragControls.dispose()
      dragControls = null
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