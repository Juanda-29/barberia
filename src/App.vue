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



      <section
        v-if="servicios.length > 0"
        class="lista"
      >

        <article
          v-for="servicio in servicios"
          :key="servicio.id"
          class="tarjeta"
        >

          <!-- CABECERA TARJETA -->

          <div class="tarjeta-header">

            <div class="cliente">

              <div class="inicial">
                {{ servicio.cliente.charAt(0).toUpperCase() }}
              </div>

              <div>

                <h3>
                  {{ servicio.cliente }}
                </h3>

                <p>
                  Atendido por {{ servicio.barbero }}
                </p>

              </div>

            </div>

            <strong class="precio">
              ${{ Number(servicio.precio).toLocaleString('es-CO') }}
            </strong>

          </div>


        

          <div class="servicio-nombre">

            <span class="mini-icono">
              ✂
            </span>

            <div>

              <small>
                SERVICIO
              </small>

              <strong>
                {{ servicio.tipoServicio }}
              </strong>

            </div>

          </div>


         

          <div class="datos">

            <div>

              <small>
                FECHA
              </small>

              <strong>
                {{ servicio.fecha }}
              </strong>

            </div>

            <div>

              <small>
                HORA
              </small>

              <strong>
                {{ servicio.hora }}
              </strong>

            </div>

          </div>



          <div class="pago">

            <div class="metodo">

              <span class="circulo-pago">
                {{ iconoPago(servicio.metodoPago) }}
              </span>

              <div>

                <small>
                  MÉTODO
                </small>

                <strong>
                  {{ servicio.metodoPago }}
                </strong>

              </div>

            </div>


            <span
              class="estado"
              :class="clasePago(servicio.estadoPago)"
            >

              <span v-if="servicio.estadoPago === 'Pagado'">
                ✓ Pagado
              </span>

              <span
                v-else-if="servicio.estadoPago === 'Pendiente'"
              >
                ! Pendiente
              </span>

              <span v-else>
                $ Fiado
              </span>

            </span>

          </div>


         

          <div
            class="calificacion"
            :class="claseCalificacion(servicio.calificacion)"
          >

            <span class="estrellas">
              {{ generarEstrellas(servicio.calificacion) }}
            </span>

            <span>

              <span
                v-if="servicio.calificacion <= 2"
              >
                Baja
              </span>

              <span
                v-else-if="servicio.calificacion === 3"
              >
                Regular
              </span>

              <span v-else>
                Buena
              </span>

            </span>

          </div>


          <div
            v-if="servicio.observaciones"
            class="observacion"
          >

            <strong>
              Observación:
            </strong>

            <p>
              {{ servicio.observaciones }}
            </p>

          </div>


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


    <div
      v-if="mostrarModal"
      class="modal-fondo"
    >

      <div class="modal">

        <div class="modal-header">

          <div>

            <h2>
              {{ editando
                ? 'Editar servicio'
                : 'Nuevo servicio'
              }}
            </h2>

            <p>
              Complete la información del servicio.
            </p>

          </div>

          <button
            class="cerrar"
            @click="cancelar"
          >
            ×
          </button>

        </div>

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

  min-height: 100vh;
  background: #f3f1ed;
  color: #292929;
  font-family:
    Arial,
    Helvetica,
    sans-serif;
}

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
  gap: 15px;
  color: white;

}

.logo-simple {

  width: 50px;
  height: 50px;
  border: 2px solid #c59a54;
  border-radius: 8px;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 25px;
  color: #c59a54;
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

</style>
