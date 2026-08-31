<script setup>
import { ref } from 'vue'
import { useLocalStorage } from '@vueuse/core'
const tiposServicio = [
  'Corte clásico',
  'Corte moderno',
  'Barba',
  'Corte + barba',
  'Cejas',
  'Tinte',
  'Otro'
]

const barberos = [
  'Don Ramiro',
  'Carlos',
  'Andrés'
]

const metodosPago = [
  'Efectivo',
  'Transferencia',
  'Tarjeta'
]

const estadosPago = [
  'Pagado',
  'Pendiente',
  'Fiado'
]

const servicios = useLocalStorage(
  'servicios-barberia',
  []
)

const mostrarModal = ref(false)
const editando = ref(false)
const idEditando = ref(null)
const errorFormulario = ref('')

function formularioVacio() {
  return {
    cliente: '',
    tipoServicio: '',
    barbero: '',
    fecha: '',
    hora: '',
    precio: '',
    metodoPago: '',
    estadoPago: '',
    calificacion: '',
    observaciones: ''
  }
}

const form = ref(formularioVacio())


function abrirModalNuevo() {
  form.value = formularioVacio()
  editando.value = false
  idEditando.value = null
  errorFormulario.value = ''
  mostrarModal.value = true
}


function editarServicio(servicio) {
  form.value = {
    cliente: servicio.cliente,
    tipoServicio: servicio.tipoServicio,
    barbero: servicio.barbero,
    fecha: servicio.fecha,
    hora: servicio.hora,
    precio: servicio.precio,
    metodoPago: servicio.metodoPago,
    estadoPago: servicio.estadoPago,
    calificacion: servicio.calificacion,
    observaciones: servicio.observaciones
  }

  idEditando.value = servicio.id
  editando.value = true
  errorFormulario.value = ''
  mostrarModal.value = true
}


function validarFormulario() {

  if (!form.value.cliente.trim()) {
    return 'Ingrese el nombre del cliente.'
  }

  if (!form.value.tipoServicio) {
    return 'Seleccione el tipo de servicio.'
  }

  if (!form.value.barbero) {
    return 'Seleccione el barbero.'
  }

  if (!form.value.fecha) {
    return 'Seleccione la fecha.'
  }

  if (!form.value.hora) {
    return 'Seleccione la hora.'
  }

  if (!form.value.precio || Number(form.value.precio) <= 0) {
    return 'Ingrese un precio válido.'
  }

  if (!form.value.metodoPago) {
    return 'Seleccione el método de pago.'
  }

  if (!form.value.estadoPago) {
    return 'Seleccione el estado del pago.'
  }

  if (
    !form.value.calificacion ||
    Number(form.value.calificacion) < 1 ||
    Number(form.value.calificacion) > 5
  ) {
    return 'Seleccione una calificación entre 1 y 5.'
  }

  return ''
}



function guardarServicio() {

  const error = validarFormulario()

  if (error) {
    errorFormulario.value = error
    return
  }

  if (editando.value) {

    const posicion = servicios.value.findIndex(
      servicio => servicio.id === idEditando.value
    )

    if (posicion !== -1) {

      servicios.value[posicion] = {
        id: idEditando.value,
        cliente: form.value.cliente,
        tipoServicio: form.value.tipoServicio,
        barbero: form.value.barbero,
        fecha: form.value.fecha,
        hora: form.value.hora,
        precio: Number(form.value.precio),
        metodoPago: form.value.metodoPago,
        estadoPago: form.value.estadoPago,
        calificacion: Number(form.value.calificacion),
        observaciones: form.value.observaciones
      }
    }

  } else {

    const nuevoServicio = {
      id: Date.now(),
      cliente: form.value.cliente,
      tipoServicio: form.value.tipoServicio,
      barbero: form.value.barbero,
      fecha: form.value.fecha,
      hora: form.value.hora,
      precio: Number(form.value.precio),
      metodoPago: form.value.metodoPago,
      estadoPago: form.value.estadoPago,
      calificacion: Number(form.value.calificacion),
      observaciones: form.value.observaciones
    }

    servicios.value.push(nuevoServicio)
  }

  cerrarModal()
}



function eliminarServicio(id) {

  const confirmar = confirm(
    '¿Está seguro de eliminar este servicio?'
  )

  if (confirmar) {

    servicios.value = servicios.value.filter(
      servicio => servicio.id !== id
    )
  }
}


function cerrarModal() {

  mostrarModal.value = false
  editando.value = false
  idEditando.value = null
  errorFormulario.value = ''
  form.value = formularioVacio()
}

function cancelar() {
  cerrarModal()
}



function totalServicios() {
  return servicios.value.length
}

function contarPagados() {

  return servicios.value.filter(
    servicio => servicio.estadoPago === 'Pagado'
  ).length
}

function contarPendientes() {

  return servicios.value.filter(
    servicio => servicio.estadoPago === 'Pendiente'
  ).length
}

function contarFiados() {

  return servicios.value.filter(
    servicio => servicio.estadoPago === 'Fiado'
  ).length
}

function totalVentas() {

  let total = 0

  servicios.value.forEach(servicio => {

    if (servicio.estadoPago === 'Pagado') {
      total += Number(servicio.precio)
    }

  })

  return total
}



function generarEstrellas(calificacion) {

  let estrellas = ''

  for (let i = 1; i <= 5; i++) {

    if (i <= Number(calificacion)) {
      estrellas += '★'
    } else {
      estrellas += '☆'
    }
  }

  return estrellas
}

function iconoPago(metodo) {

  if (metodo === 'Efectivo') {
    return 'E'
  }

  if (metodo === 'Transferencia') {
    return 'T'
  }

  return 'C'
}

function clasePago(estado) {

  if (estado === 'Pagado') {
    return 'pago-ok'
  }

  if (estado === 'Pendiente') {
    return 'pago-pendiente'
  }

  return 'pago-fiado'
}

function claseCalificacion(calificacion) {

  if (Number(calificacion) <= 2) {
    return 'calificacion-baja'
  }

  if (Number(calificacion) === 3) {
    return 'calificacion-media'
  }

  return 'calificacion-alta'
}
</script>


<template>

  <div class="pagina">

    <!-- ===============================================
         ENCABEZADO SIMPLE (sin foto, solo texto)
    ================================================ -->

    <header class="header">

      <h1>
        Barbería Don Ramiro
      </h1>

      <p>
        Control de servicios
      </p>

      <button
        class="boton-nuevo"
        @click="abrirModalNuevo"
      >
        + Nuevo servicio
      </button>

    </header>



    <main class="contenedor">

      <!-- ===============================================
           RESUMEN EN TEXTO PLANO
      ================================================ -->

      <section class="resumen">

        <p>
          Servicios: <strong>{{ totalServicios() }}</strong>
        </p>

        <p>
          Pagados: <strong>{{ contarPagados() }}</strong>
        </p>

        <p>
          Pendientes: <strong>{{ contarPendientes() }}</strong>
        </p>

        <p>
          Fiados: <strong>{{ contarFiados() }}</strong>
        </p>

        <p>
          Total vendido: <strong>${{ totalVentas().toLocaleString('es-CO') }}</strong>
        </p>

      </section>


      <hr />


      <h2>
        Servicios registrados
      </h2>


      <!-- ===============================================
           SIN REGISTROS
      ================================================ -->

      <p v-if="servicios.length === 0">
        No hay servicios registrados todavía.
      </p>


      <!-- ===============================================
           LISTA SIMPLE (sin tarjetas, sin grid)
      ================================================ -->

      <div
        v-for="servicio in servicios"
        :key="servicio.id"
        class="servicio"
      >

        <p>
          <strong>Cliente:</strong> {{ servicio.cliente }}
        </p>

        <p>
          <strong>Servicio:</strong> {{ servicio.tipoServicio }}
        </p>

        <p>
          <strong>Barbero:</strong> {{ servicio.barbero }}
        </p>

        <p>
          <strong>Fecha:</strong> {{ servicio.fecha }}
          &nbsp;
          <strong>Hora:</strong> {{ servicio.hora }}
        </p>

        <p>
          <strong>Precio:</strong> ${{ Number(servicio.precio).toLocaleString('es-CO') }}
        </p>

        <p>
          <strong>Método de pago:</strong> {{ servicio.metodoPago }}
        </p>

        <p>

          <strong>Estado:</strong>

          <span v-if="servicio.estadoPago === 'Pagado'">
            Pagado
          </span>

          <span v-else-if="servicio.estadoPago === 'Pendiente'">
            Pendiente
          </span>

          <span v-else>
            Fiado
          </span>

        </p>

        <p>
          <strong>Calificación:</strong> {{ generarEstrellas(servicio.calificacion) }}
        </p>

        <p v-if="servicio.observaciones">
          <strong>Observaciones:</strong> {{ servicio.observaciones }}
        </p>

        <button
          class="boton-editar"
          @click="editarServicio(servicio)"
        >
          Editar
        </button>

        <button
          class="boton-eliminar"
          @click="eliminarServicio(servicio.id)"
        >
          Eliminar
        </button>

        <hr />

      </div>

    </main>


    <!-- ===============================================
         MODAL SIMPLE
    ================================================ -->

    <div
      v-if="mostrarModal"
      class="modal-fondo"
    >

      <div class="modal">

        <h2>
          {{ editando ? 'Editar servicio' : 'Nuevo servicio' }}
        </h2>

        <form @submit.prevent="guardarServicio">

          <label>Nombre del cliente</label>
          <input
            v-model="form.cliente"
            type="text"
          />

          <label>Tipo de servicio</label>
          <select v-model="form.tipoServicio">
            <option value="" disabled>Seleccione</option>
            <option
              v-for="tipo in tiposServicio"
              :key="tipo"
              :value="tipo"
            >
              {{ tipo }}
            </option>
          </select>

          <label>Barbero</label>
          <select v-model="form.barbero">
            <option value="" disabled>Seleccione</option>
            <option
              v-for="barbero in barberos"
              :key="barbero"
              :value="barbero"
            >
              {{ barbero }}
            </option>
          </select>

          <label>Fecha</label>
          <input v-model="form.fecha" type="date" />

          <label>Hora</label>
          <input v-model="form.hora" type="time" />

          <label>Precio cobrado</label>
          <input
            v-model="form.precio"
            type="number"
            min="1"
          />

          <label>Método de pago</label>
          <select v-model="form.metodoPago">
            <option value="" disabled>Seleccione</option>
            <option
              v-for="metodo in metodosPago"
              :key="metodo"
              :value="metodo"
            >
              {{ metodo }}
            </option>
          </select>

          <label>Estado del pago</label>
          <select v-model="form.estadoPago">
            <option value="" disabled>Seleccione</option>
            <option
              v-for="estado in estadosPago"
              :key="estado"
              :value="estado"
            >
              {{ estado }}
            </option>
          </select>

          <label>Calificación</label>
          <select v-model="form.calificacion">
            <option value="" disabled>Seleccione</option>
            <option value="1">★☆☆☆☆</option>
            <option value="2">★★☆☆☆</option>
            <option value="3">★★★☆☆</option>
            <option value="4">★★★★☆</option>
            <option value="5">★★★★★</option>
          </select>

          <label>Observaciones (opcional)</label>
          <textarea v-model="form.observaciones"></textarea>

          <p
            v-show="errorFormulario"
            class="mensaje-error"
          >
            {{ errorFormulario }}
          </p>

          <button
            type="button"
            class="boton-cancelar"
            @click="cancelar"
          >
            Cancelar
          </button>

          <button
            type="submit"
            class="boton-guardar"
          >
            {{ editando ? 'Guardar cambios' : 'Guardar servicio' }}
          </button>

        </form>

      </div>

    </div>

  </div>

</template>


<style scoped>

* {
  box-sizing: border-box;
}

body {
  margin: 0;
}

.pagina {
  background: #ffffff;
  color: #222222;
  font-family: Arial, Helvetica, sans-serif;
  padding: 0 0 40px;
}


/* ----- encabezado simple ----- */

.header {
  background: #333333;
  color: #ffffff;
  padding: 20px;
  text-align: center;
}

.header h1 {
  margin: 0 0 5px;
  font-size: 22px;
}

.header p {
  margin: 0 0 15px;
  font-size: 14px;
}

.boton-nuevo {
  background: #555555;
  color: #ffffff;
  border: 1px solid #ffffff;
  padding: 8px 14px;
  font-size: 14px;
  cursor: pointer;
}

.boton-nuevo:hover {
  background: #666666;
}


/* ----- contenedor ----- */

.contenedor {
  max-width: 700px;
  margin: 0 auto;
  padding: 20px;
}

hr {
  border: none;
  border-top: 1px solid #cccccc;
  margin: 15px 0;
}


/* ----- resumen ----- */

.resumen p {
  margin: 4px 0;
  font-size: 14px;
}


/* ----- lista de servicios ----- */

.servicio p {
  margin: 4px 0;
  font-size: 14px;
}

.boton-editar,
.boton-eliminar {
  padding: 6px 12px;
  font-size: 13px;
  cursor: pointer;
  margin-top: 8px;
  margin-right: 8px;
}

.boton-editar {
  background: #eeeeee;
  border: 1px solid #999999;
}

.boton-eliminar {
  background: #f5dddd;
  border: 1px solid #cc8888;
}


/* ----- modal ----- */

.modal-fondo {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 15px;
}

.modal {
  background: #ffffff;
  width: 100%;
  max-width: 400px;
  max-height: 90vh;
  overflow-y: auto;
  padding: 20px;
  border: 1px solid #999999;
}

.modal h2 {
  margin: 0 0 10px;
  font-size: 18px;
}

.modal form {
  display: flex;
  flex-direction: column;
}

.modal label {
  font-size: 13px;
  font-weight: bold;
  margin-top: 10px;
}

.modal input,
.modal select,
.modal textarea {
  padding: 6px;
  font-size: 13px;
  margin-top: 4px;
  border: 1px solid #999999;
}

.modal textarea {
  height: 60px;
}

.mensaje-error {
  color: #aa0000;
  font-size: 13px;
}

.boton-cancelar,
.boton-guardar {
  margin-top: 15px;
  margin-right: 8px;
  padding: 8px 14px;
  cursor: pointer;
}

.boton-cancelar {
  background: #eeeeee;
  border: 1px solid #999999;
}

.boton-guardar {
  background: #333333;
  color: #ffffff;
  border: none;
}

</style>
