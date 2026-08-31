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

    <header class="header">

      <div class="header-imagen">

        <img
          src="https://joseppons.com/formacion/wp-content/uploads/2020/11/servicios-salon-barberia-2048x1360.jpeg"
          alt="Barbería"
        />

        <div class="imagen-overlay"></div>

        <div class="header-texto">

          <div class="titulo">
             ✂
            <div>

              <h1>
               BARBERIA DON RAMIRO
              </h1>

              <p>
                Registro de servicios
              </p>

            </div>

          </div>

          <button
            class="boton-nuevo"
            @click="abrirModalNuevo"
          >
            + Nuevo servicio
          </button>

        </div>

      </div>

    </header>



    <main class="contenedor">

      <section class="resumen">

        <div class="resumen-item">

          <span class="resumen-numero">
            {{ totalServicios() }}
          </span>

          <span class="resumen-texto">
            SERVICIOS
          </span>

        </div>


        <div class="resumen-item">

          <span class="resumen-numero">
            {{ contarPagados() }}
          </span>

          <span class="resumen-texto">
           PAGADOS
          </span>

        </div>


        <div class="resumen-item">

          <span class="resumen-numero">
            {{ contarPendientes() }}
          </span>

          <span class="resumen-texto">
            PENDIENTES
          </span>

        </div>


        <div class="resumen-item">

          <span class="resumen-numero">
            {{ contarFiados() }}
          </span>

          <span class="resumen-texto">
            FIADOS
          </span>

        </div>


        <div class="resumen-item">

          <span class="resumen-numero ventas">
            ${{ totalVentas().toLocaleString('es-CO') }}
          </span>

          <span class="resumen-texto">
            VENTAS PAGADAS
          </span>

        </div>

      </section>




      <section class="titulo-lista">

        <div>

          <h2>
            Servicios registrados
          </h2>

          <p>
            Aquí puedes consultar y administrar los servicios.
          </p>

        </div>

        <span
          v-if="servicios.length > 0"
          class="cantidad"
        >
          {{ servicios.length }}
          registrados
        </span>

      </section>


      <!-- ===============================================
           SIN REGISTROS
      ================================================ -->

      <section
        v-if="servicios.length === 0"
        class="sin-registros"
      >

        <div class="icono-vacio">
          ✂
        </div>

        <h3>
          No hay servicios registrados
        </h3>

        <p>
          Agrega el primer servicio para comenzar.
        </p>

        <button
          class="boton-nuevo"
          @click="abrirModalNuevo"
        >
          + Agregar servicio
        </button>

      </section>


      <!-- ===============================================
           LISTA (tarjeta simplificada, tipo ficha)
      ================================================ -->

      <section
        v-if="servicios.length > 0"
        class="lista"
      >

        <article
          v-for="servicio in servicios"
          :key="servicio.id"
          class="tarjeta"
        >

          <div class="tarjeta-top">

            <h3>
              {{ servicio.cliente }}
            </h3>

            <strong class="precio">
              ${{ Number(servicio.precio).toLocaleString('es-CO') }}
            </strong>

          </div>

          <p class="linea">
            {{ servicio.tipoServicio }} - Atendido por {{ servicio.barbero }}
          </p>

          <p class="linea">
            Fecha: {{ servicio.fecha }} &nbsp; Hora: {{ servicio.hora }}
          </p>

          <p class="linea">
            Pago: {{ servicio.metodoPago }} —

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

          <p class="linea">
            Calificación:
            <span class="estrellas">
              {{ generarEstrellas(servicio.calificacion) }}
            </span>
          </p>

          <p
            v-if="servicio.observaciones"
            class="observacion"
          >
            Observación: {{ servicio.observaciones }}
          </p>

          <div class="acciones">

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

          </div>

        </article>

      </section>

    </main>


    <!-- ===============================================
         MODAL (formulario simplificado)
    ================================================ -->

    <div
      v-if="mostrarModal"
      class="modal-fondo"
    >

      <div class="modal">

        <div class="modal-header">

          <h2>
            {{ editando
              ? 'Editar servicio'
              : 'Nuevo servicio'
            }}
          </h2>

          <button
            class="cerrar"
            @click="cancelar"
          >
            ×
          </button>

        </div>


        <form
          class="formulario"
          @submit.prevent="guardarServicio"
        >

          <div class="campo">

            <label>
              Nombre del cliente
            </label>

            <input
              v-model="form.cliente"
              type="text"
              placeholder="Ej. Juan Pérez"
            />

          </div>


          <div class="campo">

            <label>
              Tipo de servicio
            </label>

            <select
              v-model="form.tipoServicio"
            >

              <option
                value=""
                disabled
              >
                Seleccione
              </option>

              <option
                v-for="tipo in tiposServicio"
                :key="tipo"
                :value="tipo"
              >
                {{ tipo }}
              </option>

            </select>

          </div>


          <div class="campo">

            <label>
              Barbero
            </label>

            <select
              v-model="form.barbero"
            >

              <option
                value=""
                disabled
              >
                Seleccione
              </option>

              <option
                v-for="barbero in barberos"
                :key="barbero"
                :value="barbero"
              >
                {{ barbero }}
              </option>

            </select>

          </div>


          <div class="campo">

            <label>
              Fecha
            </label>

            <input
              v-model="form.fecha"
              type="date"
            />

          </div>


          <div class="campo">

            <label>
              Hora
            </label>

            <input
              v-model="form.hora"
              type="time"
            />

          </div>


          <div class="campo">

            <label>
              Precio cobrado
            </label>

            <input
              v-model="form.precio"
              type="number"
              min="1"
              placeholder="20000"
            />

          </div>


          <div class="campo">

            <label>
              Método de pago
            </label>

            <select
              v-model="form.metodoPago"
            >

              <option
                value=""
                disabled
              >
                Seleccione
              </option>

              <option
                v-for="metodo in metodosPago"
                :key="metodo"
                :value="metodo"
              >
                {{ metodo }}
              </option>

            </select>

          </div>


          <div class="campo">

            <label>
              Estado del pago
            </label>

            <select
              v-model="form.estadoPago"
            >

              <option
                value=""
                disabled
              >
                Seleccione
              </option>

              <option
                v-for="estado in estadosPago"
                :key="estado"
                :value="estado"
              >
                {{ estado }}
              </option>

            </select>

          </div>


          <div class="campo">

            <label>
              Calificación
            </label>

            <select
              v-model="form.calificacion"
            >

              <option
                value=""
                disabled
              >
                Seleccione
              </option>

              <option value="1">
                ★☆☆☆☆
              </option>

              <option value="2">
                ★★☆☆☆
              </option>

              <option value="3">
                ★★★☆☆
              </option>

              <option value="4">
                ★★★★☆
              </option>

              <option value="5">
                ★★★★★
              </option>

            </select>

          </div>


          <div class="campo">

            <label>
              Observaciones (opcional)
            </label>

            <textarea
              v-model="form.observaciones"
              placeholder="Escriba una observación..."
            ></textarea>

          </div>


          <div
            v-show="errorFormulario"
            class="mensaje-error"
          >
            {{ errorFormulario }}
          </div>


          <div class="botones-formulario">

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
              {{ editando
                ? 'Guardar cambios'
                : 'Guardar servicio'
              }}
            </button>

          </div>

        </form>

      </div>

    </div>

  </div>

</template>


<style scoped>

/* ======================================================
   GENERAL
====================================================== */

* {
  box-sizing: border-box;
}

body {
  margin: 0;
}

.pagina {
  min-height: 100vh;

  background: #f3f1ed;

  color: #292929;

  font-family:
    Arial,
    Helvetica,
    sans-serif;
}


/* ======================================================
   HEADER
====================================================== */

.header {
  width: 100%;
}

.header-imagen {
  height: 300px;

  position: relative;

  overflow: hidden;
}

.header-imagen img {
  width: 100%;
  height: 100%;

  object-fit: cover;
}

.imagen-overlay {
  position: absolute;

  inset: 0;

  background: rgba(0, 0, 0, .58);
}

.header-texto {
  position: absolute;

  top: 0;
  left: 0;

  width: 100%;
  height: 100%;

  display: flex;

  align-items: center;

  justify-content: space-between;

  padding: 40px;

  max-width: 1300px;

  margin: auto;

  right: 0;
}

.titulo {
  display: flex;

  align-items: center;

  gap: 35px;

  color: white;

 
}

.titulo h1 {
  margin: 0;

  font-size: 30px;
}

.titulo p {
  margin: 6px 0 0;

  color: #ddd;

  font-size: 15px;
}

.boton-nuevo {
  border: none;

  background: #c59a54;

  color: #171717;

  padding: 12px 18px;

  border-radius: 6px;

  font-size: 14px;

  font-weight: bold;

  cursor: pointer;
}

.boton-nuevo:hover {
  background: #d5aa65;
}


/* ======================================================
   CONTENEDOR
====================================================== */

.contenedor {
  width: 92%;

  max-width: 1200px;

  margin: 30px auto 60px;
}


/* ======================================================
   RESUMEN
====================================================== */

.resumen {
  display: grid;

  grid-template-columns:
    repeat(5, 1fr);

  background: white;

  border: 1px solid #ddd8cf;

  border-radius: 8px;

  overflow: hidden;

  margin-bottom: 35px;
}

.resumen-item {
  padding: 20px;

  text-align: center;

  border-right: 1px solid #e5e1da;
}

.resumen-item:last-child {
  border-right: none;
}

.resumen-numero {
  display: block;

  color: #252525;

  font-size: 23px;

  font-weight: bold;

  margin-bottom: 5px;
}

.resumen-numero.ventas {
  color: #a67a32;

  font-size: 18px;
}

.resumen-texto {
  color: #777;

  font-size: 12px;
}


/* ======================================================
   TÍTULO
====================================================== */

.titulo-lista {
  display: flex;

  align-items: center;

  justify-content: space-between;

  margin-bottom: 20px;
}

.titulo-lista h2 {
  margin: 0 0 5px;

  font-size: 24px;
}

.titulo-lista p {
  margin: 0;

  color: #777;

  font-size: 14px;
}

.cantidad {
  background: white;

  border: 1px solid #ddd8cf;

  padding: 7px 12px;

  border-radius: 20px;

  color: #777;

  font-size: 12px;
}


/* ======================================================
   SIN REGISTROS
====================================================== */

.sin-registros {
  background: white;

  border: 1px solid #ddd8cf;

  border-radius: 8px;

  padding: 60px 20px;

  text-align: center;
}

.icono-vacio {
  width: 60px;
  height: 60px;

  margin: 0 auto 15px;

  display: flex;

  align-items: center;

  justify-content: center;

  border-radius: 50%;

  background: #f0e8da;

  color: #a67a32;

  font-size: 25px;
}

.sin-registros h3 {
  margin: 0 0 7px;
}

.sin-registros p {
  color: #777;

  margin: 0 0 20px;
}


/* ======================================================
   LISTA
====================================================== */

.lista {
  display: grid;

  grid-template-columns:
    repeat(3, 1fr);

  gap: 18px;
}


/* ======================================================
   TARJETA - versión simple, tipo ficha de texto
====================================================== */

.tarjeta {
  background: white;

  border: 1px solid #ddd8cf;

  border-radius: 6px;

  padding: 16px;
}

.tarjeta-top {
  display: flex;

  align-items: center;

  justify-content: space-between;

  gap: 10px;

  padding-bottom: 10px;

  margin-bottom: 8px;

  border-bottom: 1px solid #eeeae3;
}

.tarjeta-top h3 {
  margin: 0;

  font-size: 15px;
}

.precio {
  color: #a67a32;

  font-size: 15px;

  white-space: nowrap;
}

.linea {
  margin: 6px 0;

  font-size: 13px;

  color: #444;
}

.estrellas {
  color: #c59a54;

  letter-spacing: 1px;
}

.observacion {
  background: #f7f5f1;

  padding: 8px;

  margin: 10px 0;

  font-size: 12px;

  color: #666;

  border-radius: 4px;
}


/* ======================================================
   ACCIONES
====================================================== */

.acciones {
  display: grid;

  grid-template-columns: 1fr 1fr;

  gap: 8px;

  margin-top: 10px;
}

.boton-editar,
.boton-eliminar {
  padding: 9px;

  border-radius: 5px;

  cursor: pointer;

  font-size: 12px;

  font-weight: bold;
}

.boton-editar {
  border: 1px solid #ccc;

  background: white;

  color: #555;
}

.boton-eliminar {
  border: 1px solid #e0c3c3;

  background: #fff7f7;

  color: #a05252;
}


/* ======================================================
   MODAL - versión simple
====================================================== */

.modal-fondo {
  position: fixed;

  inset: 0;

  background: rgba(0, 0, 0, .6);

  display: flex;

  align-items: center;

  justify-content: center;

  padding: 20px;

  z-index: 100;
}

.modal {
  width: 100%;

  max-width: 500px;

  max-height: 92vh;

  overflow-y: auto;

  background: white;

  border-radius: 6px;
}

.modal-header {
  display: flex;

  justify-content: space-between;

  align-items: center;

  padding: 16px 18px;

  border-bottom: 1px solid #ddd;
}

.modal-header h2 {
  margin: 0;

  font-size: 18px;
}

.cerrar {
  width: 28px;
  height: 28px;

  border: none;

  background: #eee;

  border-radius: 50%;

  font-size: 18px;

  cursor: pointer;
}


/* ======================================================
   FORMULARIO
====================================================== */

.formulario {
  padding: 18px;

  display: flex;

  flex-direction: column;

  gap: 12px;
}

.campo {
  display: flex;

  flex-direction: column;

  gap: 5px;
}

.campo label {
  font-size: 12px;

  font-weight: bold;

  color: #555;
}

.campo input,
.campo select,
.campo textarea {
  width: 100%;

  padding: 9px;

  border: 1px solid #d5d2cd;

  border-radius: 4px;

  background: white;

  color: #333;

  outline: none;

  font-size: 13px;
}

.campo input:focus,
.campo select:focus,
.campo textarea:focus {
  border-color: #b48a48;
}

.campo textarea {
  height: 70px;

  resize: vertical;
}


/* ======================================================
   ERROR
====================================================== */

.mensaje-error {
  padding: 9px;

  background: #f9e6e6;

  border: 1px solid #eccaca;

  color: #a04d4d;

  border-radius: 4px;

  font-size: 12px;
}


/* ======================================================
   BOTONES FORMULARIO
====================================================== */

.botones-formulario {
  display: grid;

  grid-template-columns: 1fr 1fr;

  gap: 10px;

  margin-top: 5px;
}

.boton-cancelar,
.boton-guardar {
  padding: 10px;

  border-radius: 4px;

  cursor: pointer;

  font-weight: bold;
}

.boton-cancelar {
  border: 1px solid #ccc;

  background: white;

  color: #555;
}

.boton-guardar {
  border: none;

  background: #282828;

  color: white;
}

.boton-guardar:hover {
  background: #3b3b3b;
}


/* ======================================================
   RESPONSIVE
====================================================== */

@media (max-width: 950px) {

  .lista {
    grid-template-columns:
      repeat(2, 1fr);
  }

  .resumen {
    grid-template-columns:
      repeat(3, 1fr);
  }

  .resumen-item:nth-child(3) {
    border-right: none;
  }

}


@media (max-width: 650px) {

  .header-imagen {
    height: 260px;
  }

  .header-texto {
    padding: 25px;

    flex-direction: column;

    align-items: flex-start;

    justify-content: center;

    gap: 25px;
  }

  .titulo h1 {
    font-size: 23px;
  }

  .boton-nuevo {
    width: 100%;
  }

  .resumen {
    grid-template-columns:
      repeat(2, 1fr);
  }

  .resumen-item {
    border-bottom: 1px solid #e5e1da;
  }

  .lista {
    grid-template-columns: 1fr;
  }

  .titulo-lista {
    align-items: flex-start;

    flex-direction: column;

    gap: 10px;
  }

}


@media (max-width: 480px) {

  .contenedor {
    width: 94%;
  }

  .resumen-numero {
    font-size: 20px;
  }

  .resumen-numero.ventas {
    font-size: 15px;
  }

  .modal-fondo {
    padding: 10px;
  }

  .botones-formulario {
    grid-template-columns: 1fr;
  }

}

</style>
