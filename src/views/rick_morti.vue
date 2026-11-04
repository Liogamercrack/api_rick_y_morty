
<template>
    <main>
        <h1>Personajes de Rick and Morty</h1>

        <!-- Campo de texto para filtrar -->
        <div class="buscador-container">
            <input
                type="text"
                id="input_buscar"
                placeholder="Buscar personaje por nombre..."
                v-model="textoBusqueda"
            >
        </div>

        <div id="mostrar_personajes">

            <p v-if="buscarPersonajes().length === 0">
                No se encontraron personajes.
            </p>

            <div
                class="card"
                v-for="personaje in buscarPersonajes()"
                :key="personaje.id"
            >
                <img :src="personaje.image">

                <h3>{{ personaje.name }}</h3>

                <p>Especie: {{ personaje.species }}</p>

                <p>Estado: {{ personaje.status }}</p>

                <p>Género: {{ personaje.gender }}</p>
            </div>

        </div>
    </main>
</template>

<script setup>
import { ref } from "vue";

// Guardaremos los personajes globalmente para no volver a hacer fetch en cada búsqueda
const listaPersonajes = ref([]);

const textoBusqueda = ref("");

async function mostrar() {
    try {
        let datos = await fetch(
            "https://rickandmortyapi.com/api/character"
        );

        let respuesta = await datos.json();

        // Guardamos los resultados
        listaPersonajes.value = respuesta.results;

    } catch(error) {
        alert("Error de conexión: " + error);
    }
}

// Función encargada de filtrar los personajes
function buscarPersonajes() {

    let texto = textoBusqueda.value.toLowerCase();

    // Filtramos la lista original comparando los nombres
    let personajesFiltrados = listaPersonajes.value.filter(function(personaje) {
        return personaje.name.toLowerCase().includes(texto);
    });

    // Devolvemos los personajes que coinciden
    return personajesFiltrados;
}

// Inicializamos la carga de datos
mostrar();

</script>

<style scoped>
body {
    font-family: Arial, sans-serif;
    background-color: #222;
    color: white;
    margin: 0;
    padding: 20px;
}

h1 {
    text-align: center;
}

/* Estilos para el contenedor del buscador */
.buscador-container {
    text-align: center;
    margin-bottom: 25px;
}

#input_buscar {
    padding: 10px 15px;
    width: 80%;
    max-width: 400px;
    font-size: 16px;
    border-radius: 8px;
    border: 2px solid #555;
    background-color: #333;
    color: white;
    outline: none;
    transition: border-color 0.3s;
}

#input_buscar:focus {
    border-color: #00b5cc;
}

#mostrar_personajes {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    gap: 20px;
}

.card {
    width: 220px;
    background-color: #333;
    border-radius: 10px;
    padding: 15px;
    text-align: center;
}

.card img {
    width: 100%;
    border-radius: 10px;
}

.card h3 {
    margin-bottom: 10px;
}

.card p {
    margin: 5px;
}
</style>
