<template>
  <main class="contenedor">
    <header class="encabezado">
      <div class="icono-dolar">$</div>
      <h1>Api Dólar</h1>
      <p>Consultá cotizaciones y simulá tus operaciones</p>
    </header>

    <!-- Cotizaciones -->
    <section class="seccion">
      <h2>Cotizaciones actuales</h2>

      <p v-if="cargando" class="mensaje">Cargando cotizaciones...</p>

      <p v-else-if="errorApi" class="mensaje error">
        {{ errorApi }}
      </p>

      <div v-else class="tarjetas">
        <article v-for="d in dolar" :key="d.nombre" class="tarjeta">
          <h3>{{ d.nombre }}</h3>

          <div class="precio">
            <span>Compra</span>
            <strong>{{ moneda(d.compra) }}</strong>
          </div>

          <div class="precio">
            <span>Venta</span>
            <strong>{{ moneda(d.venta) }}</strong>
          </div>
        </article>
      </div>
    </section>

    <!-- Formulario -->
    <section class="seccion">
      <h2>Comprar o vender dólares</h2>

      <form class="formulario" @submit.prevent="operacion">
        <div class="campo">
          <label for="tipo">Tipo de dólar</label>

          <select id="tipo" v-model="tipoDolar" required>
            <option value="Blue">Dólar Blue</option>
            <option value="Bolsa">Dólar Bolsa</option>
            <option value="Mayorista">Dólar Mayorista</option>
            <option value="Cripto">Dólar Cripto</option>
            <option value="CCL">Dólar CCL</option>
            <option value="Tarjeta">Dólar Tarjeta</option>
          </select>
        </div>

        <div class="campo">
          <label for="tipoOperacion">Operación</label>

          <select id="tipoOperacion" v-model="opcion" required>
            <option value="Comprar">Comprar dólares</option>
            <option value="Vender">Vender dólares</option>
          </select>
        </div>

        <div class="campo">
          <label for="monto">
            {{
              opcion === "Comprar" ? "Monto en pesos argentinos" : "Cantidad de dólares a vender"
            }}
          </label>

          <input
            id="monto"
            v-model.number="monto"
            type="number"
            min="0.01"
            step="0.01"
            placeholder="Ej: 100000"
            required
          />
        </div>

        <button type="submit" :disabled="cargando || !!errorApi">Guardar operación</button>
      </form>

      <p class="nota">
        Simulación educativa. Los valores dependen de la cotización recibida desde la API.
      </p>
    </section>

    <!-- Resultado -->
    <section v-if="resultado" class="resultado">
      <h2>Resultado de la operación</h2>
      <p>{{ resultado.descripcion }}</p>
      <strong>{{ moneda(resultado.total) }}</strong>
      <p class="detalle">{{ resultado.detalle }}</p>
    </section>

    <!-- Historial -->
    <section class="seccion">
      <div class="titulo-historial">
        <h2>Historial de operaciones</h2>
        <button
          v-if="operaciones.length"
          class="boton-limpiar"
          type="button"
          @click="operaciones = []"
        >
          Limpiar
        </button>
      </div>

      <p v-if="!operaciones.length" class="mensaje">Todavía no guardaste ninguna operación.</p>

      <div v-else class="tabla-contenedor">
        <table>
          <thead>
            <tr>
              <th>Tipo de dólar</th>
              <th>Operación</th>
              <th>Monto ingresado</th>
              <th>Resultado</th>
            </tr>
          </thead>

          <tbody>
            <tr v-for="op in operaciones" :key="op.id">
              <td>{{ op.tipo }}</td>
              <td>
                <span :class="op.operacion === 'Comprar' ? 'etiqueta comprar' : 'etiqueta vender'">
                  {{ op.operacion }}
                </span>
              </td>
              <td>{{ moneda(op.monto) }}</td>
              <td>{{ moneda(op.total) }}</td>
            </tr>
          </tbody>
        </table>
      </div>
    </section>

    <footer>Datos obtenidos de DolarAPI. Las cotizaciones pueden cambiar.</footer>
  </main>
</template>

<script setup>
import { ref } from "vue";

const dolar = ref([]);
const cargando = ref(true);
const errorApi = ref("");

const tipoDolar = ref("Blue");
const opcion = ref("Comprar");
const monto = ref(null);

const operaciones = ref([]);
const resultado = ref(null);

async function mostrar() {
  cargando.value = true;
  errorApi.value = "";

  try {
    const respuesta = await fetch("https://dolarapi.com/v1/dolares");

    if (!respuesta.ok) {
      throw new Error("No se pudieron cargar las cotizaciones.");
    }

    const datos = await respuesta.json();
    dolar.value = datos;
  } catch (error) {
    errorApi.value = "Error al cargar los dólares. Revisá tu conexión e intentá nuevamente.";
    console.error(error);
  } finally {
    cargando.value = false;
  }
}

function moneda(valor) {
  return new Intl.NumberFormat("es-AR", {
    style: "currency",
    currency: "ARS",
    minimumFractionDigits: 2,
    maximumFractionDigits: 2,
  }).format(valor ?? 0);
}

function buscarDolar(tipo) {
  const nombres = {
    Blue: ["blue"],
    Bolsa: ["bolsa"],
    Mayorista: ["mayorista"],
    Cripto: ["cripto"],
    CCL: ["contado con liquidación", "contado con liquidacion"],
    Tarjeta: ["tarjeta"],
  };

  const opciones = nombres[tipo] || [];

  return dolar.value.find((d) => {
    const nombre = d.nombre
      .normalize("NFD")
      .replace(/[\u0300-\u036f]/g, "")
      .toLowerCase();

    return opciones.some((op) => {
      const buscado = op
        .normalize("NFD")
        .replace(/[\u0300-\u036f]/g, "")
        .toLowerCase();

      return nombre.includes(buscado);
    });
  });
}

function operacion() {
  const cantidad = Number(monto.value);

  if (!Number.isFinite(cantidad) || cantidad <= 0) {
    alert("Ingresá un monto mayor que cero.");
    return;
  }

  const cotizacion = buscarDolar(tipoDolar.value);

  if (!cotizacion) {
    alert("No se encontró ese tipo de dólar en la API.");
    return;
  }

  // Al comprar, la casa vende dólares.
  // Al vender, la casa compra dólares.
  const precio = opcion.value === "Comprar" ? Number(cotizacion.venta) : Number(cotizacion.compra);

  if (!Number.isFinite(precio) || precio <= 0) {
    alert("La cotización no es válida.");
    return;
  }

  const total = opcion.value === "Comprar" ? cantidad / precio : cantidad * precio;

  const nuevaOperacion = {
    id: Date.now(),
    tipo: cotizacion.nombre,
    operacion: opcion.value,
    monto: cantidad,
    total: Number(total.toFixed(2)),
  };

  operaciones.value.unshift(nuevaOperacion);

  resultado.value = {
    descripcion:
      opcion.value === "Comprar"
        ? "Dólares que podrías comprar:"
        : "Pesos que recibirías por la venta:",
    total: nuevaOperacion.total,
    detalle:
      opcion.value === "Comprar"
        ? `Cotización de venta: ${moneda(precio)} por dólar.`
        : `Cotización de compra: ${moneda(precio)} por dólar.`,
  };
}

mostrar();
</script>

<style scoped>
* {
  box-sizing: border-box;
}

.contenedor {
  min-height: 100vh;
  padding: 35px 20px;
  background: #f1f5f9;
  color: #1e293b;
  font-family: Arial, Helvetica, sans-serif;
}

.encabezado {
  max-width: 1100px;
  margin: 0 auto 35px;
  padding: 35px 20px;
  text-align: center;
  color: white;
  background: linear-gradient(135deg, #0f766e, #164e63);
  border-radius: 18px;
  box-shadow: 0 10px 25px #0f766e25;
}

.icono-dolar {
  display: grid;
  place-items: center;
  width: 60px;
  height: 60px;
  margin: 0 auto 12px;
  border: 2px solid #99f6e4;
  border-radius: 50%;
  color: #99f6e4;
  font-size: 35px;
  font-weight: bold;
}

.encabezado h1 {
  margin: 0 0 10px;
  font-size: clamp(28px, 4vw, 40px);
}

.encabezado p {
  margin: 0;
  color: #ccfbf1;
}

.seccion,
.resultado {
  max-width: 1100px;
  margin: 0 auto 28px;
  padding: 25px;
  background: white;
  border: 1px solid #e2e8f0;
  border-radius: 16px;
  box-shadow: 0 5px 18px #0f172a08;
}

.seccion h2,
.resultado h2 {
  margin: 0 0 22px;
  font-size: 22px;
  color: #0f766e;
}

.tarjetas {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(210px, 1fr));
  gap: 16px;
}

.tarjeta {
  padding: 20px;
  border: 1px solid #dbeafe;
  border-radius: 12px;
  background: #f8fafc;
  transition:
    transform 0.2s,
    box-shadow 0.2s;
}

.tarjeta:hover {
  transform: translateY(-3px);
  box-shadow: 0 8px 18px #0f172a12;
}

.tarjeta h3 {
  margin: 0 0 18px;
  color: #0f766e;
  font-size: 17px;
}

.precio {
  display: flex;
  justify-content: space-between;
  gap: 12px;
  padding: 10px 0;
  border-top: 1px solid #e2e8f0;
}

.precio span {
  color: #64748b;
}

.precio strong {
  color: #1e293b;
}

.formulario {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 20px;
}

.campo {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.campo label {
  font-size: 14px;
  font-weight: bold;
  color: #334155;
}

.campo select,
.campo input {
  width: 100%;
  min-height: 46px;
  padding: 12px;
  border: 1px solid #cbd5e1;
  border-radius: 9px;
  background: white;
  color: #1e293b;
  font: inherit;
  outline: none;
  transition:
    border-color 0.2s,
    box-shadow 0.2s;
}

.campo select:focus,
.campo input:focus {
  border-color: #0d9488;
  box-shadow: 0 0 0 3px #0d948820;
}

.formulario > button {
  grid-column: 1 / -1;
  min-height: 48px;
  padding: 12px 18px;
  border: none;
  border-radius: 9px;
  background: #0f766e;
  color: white;
  font-size: 16px;
  font-weight: bold;
  cursor: pointer;
  transition: background 0.2s;
}

.formulario > button:hover {
  background: #115e59;
}

.formulario > button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.nota {
  margin: 16px 0 0;
  color: #64748b;
  font-size: 13px;
  line-height: 1.5;
}

.resultado {
  border-left: 5px solid #0d9488;
  background: #f0fdfa;
}

.resultado h2 {
  margin-bottom: 12px;
}

.resultado > strong {
  display: block;
  margin: 12px 0;
  color: #0f766e;
  font-size: clamp(25px, 4vw, 34px);
  overflow-wrap: anywhere;
}

.detalle {
  color: #64748b;
  font-size: 14px;
}

.titulo-historial {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 15px;
}

.titulo-historial h2 {
  margin-bottom: 20px;
}

.boton-limpiar {
  margin-bottom: 20px;
  padding: 9px 13px;
  border: 1px solid #fecaca;
  border-radius: 8px;
  background: #fff1f2;
  color: #be123c;
  cursor: pointer;
}

.boton-limpiar:hover {
  background: #ffe4e6;
}

.tabla-contenedor {
  width: 100%;
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
  text-align: left;
  font-size: 14px;
}

thead {
  background: #0f766e;
  color: white;
}

th,
td {
  padding: 14px 12px;
  border-bottom: 1px solid #e2e8f0;
  white-space: nowrap;
}

tbody tr:nth-child(even) {
  background: #f8fafc;
}

tbody tr:hover {
  background: #f0fdfa;
}

.etiqueta {
  display: inline-block;
  padding: 5px 9px;
  border-radius: 20px;
  font-size: 12px;
  font-weight: bold;
}

.comprar {
  background: #d1fae5;
  color: #065f46;
}

.vender {
  background: #dbeafe;
  color: #1d4ed8;
}

.mensaje {
  padding: 18px;
  border-radius: 10px;
  background: #f8fafc;
  color: #64748b;
  text-align: center;
}

.error {
  background: #fef2f2;
  color: #b91c1c;
}

footer {
  max-width: 1100px;
  margin: 30px auto 0;
  color: #64748b;
  text-align: center;
  font-size: 13px;
}

@media (max-width: 650px) {
  .contenedor {
    padding: 20px 12px;
  }

  .encabezado {
    padding: 28px 16px;
  }

  .seccion,
  .resultado {
    padding: 18px;
  }

  .formulario {
    grid-template-columns: 1fr;
  }

  .formulario > button {
    grid-column: auto;
  }

  .tarjetas {
    grid-template-columns: 1fr;
  }

  .titulo-historial {
    align-items: flex-start;
  }
}
</style>
