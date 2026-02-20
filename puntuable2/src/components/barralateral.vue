<template>
    <div class="barra-lateral">
        <h1>MUEBLES DISPONIBLES</h1>
            <img 
                src="../assets/silla.png" 
                alt="Silla" 
                width="100" 
                draggable="true"
                @dragstart="startDrag($event, 'silla')"
                class="draggable-item">
            <img 
                src="../assets/mesa.png" 
                alt="Mesa" 
                width="100" 
                draggable="true"
                @dragstart="startDrag($event, 'mesa')"
                class="draggable-item">
            <img 
                src="../assets/cama.png" 
                alt="Cama" 
                width="100" 
                draggable="true"
                @dragstart="startDrag($event, 'cama')"
                class="draggable-item">
    </div>
</template>

<script setup lang="ts">
const startDrag = (event: DragEvent, tipo: string) => {
  if (event.dataTransfer) {
    event.dataTransfer.effectAllowed = 'copy'
    event.dataTransfer.setData('tipo-mueble', tipo)
    console.log('Arrastrando:', tipo)
  }
}
</script>

<style scoped>
.barra-lateral {
    position: fixed;
    left: 0;
    top: 0;
    width: 250px;
    height: 100vh;
    background-color: #fff;
    padding: 20px;
    z-index: 1000;
    overflow-y: auto;
    display: flex;
    flex-direction: column;
    align-items: center;
}
.barra-lateral h1 {
    font-size: 18px;
    margin-bottom: 20px;
    font-family: 'Courier New', Courier, monospace;
}

.draggable-item {
    cursor: grab;
    margin: 10px 0;
    transition: transform 0.2s;
}

.draggable-item:hover {
    transform: scale(1.05);
}

.draggable-item:active {
    cursor: grabbing;
}
</style>