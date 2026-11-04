<template>
<form action="">
    nombre <input type="txt" v-model="nombre" >
    apellido <input type="txt" v-model="apellido">
    edad <input type="txt" v-model="edad" >
    <input type="button" value="Guardar" @click="guardar()">
</form>

<h1>Introduccion Vue</h1>
<h2 >Mi nombre es {{ nombre }} y Mi apellido es {{ apellido }}</h2>
<h3 >Mi edad es {{ edad }}</h3>

<h2>Calculadora</h2>
<form action="">
    Num1 <input type="text" v-model="n1">
    Num2 <input type="text" v-model="n2">
    <input type="button" value="+" @click="sumar()">
    <input type="button" value="-" @click="restar()">
    <input type="button" value="*" @click="multiplicacion()">
    <input type="button" value="/" @click="divicion()">
</form>

<h2>La suma es {{ Resultado }}</h2>
<h3>Listado de estudiantes FOR UNIDIMENSIONAL</h3>
<ul>
    <li v-for="i in nombres" :key ="i">{{ i }} </li>
</ul>

<p>el nombre es {{ nombres[2] }}</p>
<p>El vector estudiantes es estudiantes </p>

<p>La nota del estudiante {{estudiantes[0].nombre}} {{estudiantes[0].apellido}} es {{ estudiantes[0].nota }}</p>

<h2>For de obejtos</h2>
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



<script setup >
import {ref} from 'vue'

var nombre = ref("")
var apellido =ref("")
var edad =ref("")
var n1 = ref("")
var n2 =ref("")
var Resultado =ref("")
var nombres = ["Adriel","David","pedro","mateo"]
var estudiantes = [
    {nombre: "Lucas", apellido: "Gonzalez", materia: "Matematica", nota: 8},
    {nombre: "Sofia", apellido: "Martinez", materia: "Programacion", nota: 9},
    {nombre: "Mateo", apellido: "Rodriguez", materia: "Base de Datos", nota: 7},
    {nombre: "Camila", apellido: "Fernandez", materia: "Ingles", nota: 10}
]


function sumar(){
    Resultado.value = Number(n1.value) + Number(n2.value)
}
function restar(){
    Resultado.value = Number(n1.value) - Number(n2.value)
}
function multiplicacion(){
    Resultado.value = Number(n1.value) * Number(n2.value)
}
function divicion(){
    Resultado.value = Number(n1.value) / Number(n2.value)
}



</script>