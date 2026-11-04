<template>
  <h1>Listado de Estudiantes</h1>
  <table>
    <tr>
      <th>Nombre</th>
      <th>Apellido</th>
      <th>Materia</th>
      <th>Nota</th>
    </tr>
    <tr v-for="i in estudiantes" :key="i">
      <th>{{ i.nombre }}</th>
      <th>{{ i.apellido }}</th>
      <th>{{ i.materia }}</th>
      <th>{{ i.nota }}</th>
    </tr>
  </table>
</template>

<script setup>
import { ref } from "vue";

var estudiantes = ref([]);

async function mostrar() {
  try {
    let datos = await fetch("./estudiantes.json");
    estudiantes.value = await datos.json();
  } catch (error) {
    alert(error);
  }
}
mostrar();
</script>



<style scoped>
/* Fondo y estilo general */
* {
  box-sizing: border-box;
}

h1 {
  text-align: center;
  color: #1e3a8a;
  font-family: Arial, sans-serif;
  margin: 30px 0;
}

/* Tabla */
table {
  width: 80%;
  margin: 0 auto;
  border-collapse: collapse;
  font-family: Arial, sans-serif;
  background-color: white;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.12);
  border-radius: 8px;
  overflow: hidden;
}

/* Encabezados */
th {
  padding: 14px 18px;
  text-align: left;
  border-bottom: 1px solid #e5e7eb;
}

tr:first-child {
  background-color: #2563eb;
  color: white;
}

/* Filas de estudiantes */
tr:not(:first-child) {
  color: #374151;
  transition: background-color 0.2s ease;
}

tr:not(:first-child):nth-child(even) {
  background-color: #f3f6fb;
}

tr:not(:first-child):hover {
  background-color: #dbeafe;
}

/* Adaptación a celulares */
@media (max-width: 600px) {
  table {
    width: 95%;
    font-size: 13px;
  }

  th {
    padding: 10px 6px;
    overflow-wrap: anywhere;
  }

  h1 {
    font-size: 24px;
  }
}
</style>

