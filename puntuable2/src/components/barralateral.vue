<template>
    <div class="barra-lateral">
        <nav>
            <ul>
                <li><a href="#inicio">Inicio</a></li>
                <li><a href="#servicios">Servicios</a></li>
                <li><a href="#contacto">Contacto</a></li>
            </ul>
        </nav>
    </div>
</template>

<script>
export default {
    name: 'BarraLateral'
}
</script>

<style scoped>
.barra-lateral {
    position: fixed;
    left: 0;
    top: 0;
    width: 250px;
    height: 100vh;
    background-color: #333;
    padding: 20px;
    z-index: 1000;
    overflow-y: auto;
}

nav ul {
    list-style: none;
    padding: 0;
    margin: 0;
}

nav ul li {
    margin: 15px 0;
}

nav ul li a {
    color: #fff;
    text-decoration: none;
    font-size: 16px;
    display: block;
    padding: 10px;
    border-radius: 4px;
    transition: background-color 0.3s;
}

nav ul li a:hover {
    background-color: #555;
}
</style>