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

        <!-- NUEVO: sección de personajes guardados -->
        <div v-if="personajesGuardados.length > 0" class="guardados-container">
            <h2>Personajes guardados ({{ personajesGuardados.length }})</h2>
            <div class="guardados-lista">
                <div
                    class="card"
                    v-for="personaje in personajesGuardados"
                    :key="personaje.id"
                >
                    <img :src="personaje.image">
                    <h3>{{ personaje.name }}</h3>
                    <button class="btn-guardar guardado" @click="quitarPersonaje(personaje.id)">
                        Quitar
                    </button>
                </div>
            </div>
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

                <!-- NUEVO: botón para guardar -->
                <button
                    class="btn-guardar"
                    :class="{ guardado: estaGuardado(personaje.id) }"
                    @click="guardarPersonaje(personaje)"
                >
                    {{ estaGuardado(personaje.id) ? "Guardado ✓" : "Guardar" }}
                </button>
            </div>

        </div>
    </main>
</template>

<script setup>
import { ref } from "vue";

// Guardaremos los personajes globalmente para no volver a hacer fetch en cada búsqueda
const listaPersonajes = ref([]);

const textoBusqueda = ref("");

// NUEVO: lista de personajes guardados (se recupera de localStorage al iniciar)
const personajesGuardados = ref([]);

try {
    const guardadosPrevios = localStorage.getItem("personajesGuardados");
    if (guardadosPrevios) {
        personajesGuardados.value = JSON.parse(guardadosPrevios);
    }
} catch (error) {
    personajesGuardados.value = [];
}

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

// NUEVO: indica si un personaje ya está guardado
function estaGuardado(id) {
    return personajesGuardados.value.some(function(p) {
        return p.id === id;
    });
}

// NUEVO: guarda un personaje (si ya estaba guardado, no hace nada)
function guardarPersonaje(personaje) {
    if (estaGuardado(personaje.id)) {
        return;
    }
    personajesGuardados.value.push(personaje);
    actualizarAlmacenamiento();
}

// NUEVO: quita un personaje de la lista de guardados
function quitarPersonaje(id) {
    personajesGuardados.value = personajesGuardados.value.filter(function(p) {
        return p.id !== id;
    });
    actualizarAlmacenamiento();
}

// NUEVO: persiste la lista en localStorage
function actualizarAlmacenamiento() {
    try {
        localStorage.setItem(
            "personajesGuardados",
            JSON.stringify(personajesGuardados.value)
        );
    } catch (error) {
        alert("No se pudo guardar: " + error);
    }
}

// Inicializamos la carga de datos
mostrar();

</script>

<style >
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

/* NUEVO: estilos del botón guardar y la sección de guardados */
.btn-guardar {
    margin-top: 10px;
    padding: 8px 16px;
    font-size: 14px;
    border: none;
    border-radius: 8px;
    background-color: #00b5cc;
    color: white;
    cursor: pointer;
    transition: background-color 0.3s;
}

.btn-guardar:hover {
    background-color: #008fa1;
}

.btn-guardar.guardado {
    background-color: #555;
}

.guardados-container {
    margin-bottom: 30px;
    padding: 15px;
    border: 2px dashed #00b5cc;
    border-radius: 10px;
}

.guardados-container h2 {
    text-align: center;
    margin-top: 0;
}

.guardados-lista {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    gap: 20px;
}
</style>