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
          src="https://www.beautymarket.es/imagen/min35798.jpg"
          alt="Barbería"
        />

        <div class="imagen-overlay"></div>

        <div class="header-texto">

          <div class="titulo">
            
            <div>

              <h1>
              ✂ BARBERIA DON RAMIRO
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
            Aquí puedes consultar y administrar los servicios
          </p>

        </div>

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
              placeholder="Ej. Juan Castro"
            />

          </div>


          

          <div class="fila">

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

          </div>



          <div class="fila">

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


        

          <div class="fila">

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
              Observaciones
              <span>(opcional)</span>
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

  gap: 35px;

  color: rgb(252, 250, 250);

 
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



.contenedor {
  width: 92%;

  max-width: 1200px;

  margin: 30px auto 60px;
}


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




.lista {
  display: grid;

  grid-template-columns:
    repeat(3, 1fr);

  gap: 18px;
}



.tarjeta {
  background: white;

  border: 1px solid #ddd8cf;

  border-radius: 8px;

  padding: 18px;

  transition: .2s;
}

.tarjeta:hover {
  border-color: #c59a54;

  box-shadow:
    0 4px 15px rgba(0, 0, 0, .06);
}




.tarjeta-header {
  display: flex;

  align-items: center;

  justify-content: space-between;

  gap: 10px;

  padding-bottom: 15px;

  border-bottom: 1px solid #eeeae3;
}

.cliente {
  display: flex;

  align-items: center;

  gap: 10px;
}

.inicial {
  width: 40px;
  height: 40px;

  display: flex;

  align-items: center;

  justify-content: center;

  border-radius: 50%;

  background: #282828;

  color: #c59a54;

  font-weight: bold;
}

.cliente h3 {
  margin: 0;

  font-size: 15px;
}

.cliente p {
  margin: 4px 0 0;

  color: #888;

  font-size: 11px;
}

.precio {
  color: #a67a32;

  font-size: 16px;

  white-space: nowrap;
}



.servicio-nombre {
  display: flex;

  align-items: center;

  gap: 10px;

  padding: 14px 0;
}

.mini-icono {
  width: 35px;
  height: 35px;

  display: flex;

  align-items: center;

  justify-content: center;

  background: #f3eee6;

  color: #a67a32;

  border-radius: 6px;
}

.servicio-nombre small,
.datos small,
.metodo small {
  display: block;

  color: #999;

  font-size: 9px;

  margin-bottom: 3px;

  letter-spacing: .5px;
}

.servicio-nombre strong {
  font-size: 13px;
}




.datos {
  display: grid;

  grid-template-columns: 1fr 1fr;

  gap: 10px;

  padding-bottom: 14px;
}

.datos strong {
  font-size: 12px;
}


.pago {
  display: flex;

  align-items: center;

  justify-content: space-between;

  gap: 10px;

  padding: 13px 0;

  border-top: 1px solid #eeeae3;

  border-bottom: 1px solid #eeeae3;
}

.metodo {
  display: flex;

  align-items: center;

  gap: 8px;
}

.circulo-pago {
  width: 30px;
  height: 30px;

  display: flex;

  align-items: center;

  justify-content: center;

  border-radius: 50%;

  background: #292929;

  color: #c59a54;

  font-size: 11px;

  font-weight: bold;
}

.metodo strong {
  font-size: 11px;
}

.estado {
  padding: 5px 8px;

  border-radius: 4px;

  font-size: 10px;

  font-weight: bold;
}

.pago-ok {
  background: #e5f3e7;

  color: #3f814b;
}

.pago-pendiente {
  background: #fff1d9;

  color: #9a6b24;
}

.pago-fiado {
  background: #f8e4e4;

  color: #9c4b4b;
}



.calificacion {
  display: flex;

  justify-content: space-between;

  align-items: center;

  padding: 12px 0;

  font-size: 11px;
}

.estrellas {
  font-size: 15px;

  letter-spacing: 1px;
}

.calificacion-alta .estrellas {
  color: #c59a54;
}

.calificacion-media .estrellas {
  color: #c28a35;
}

.calificacion-baja .estrellas {
  color: #b65353;
}



.observacion {
  background: #f7f5f1;

  border-left: 3px solid #c59a54;

  padding: 9px;

  margin-bottom: 13px;

  font-size: 11px;
}

.observacion strong {
  font-size: 10px;
}

.observacion p {
  margin: 4px 0 0;

  color: #777;

  line-height: 1.4;
}



.acciones {
  display: grid;

  grid-template-columns: 1fr 1fr;

  gap: 8px;
}

.boton-editar,
.boton-eliminar {
  padding: 9px;

  border-radius: 5px;

  cursor: pointer;

  font-size: 11px;

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




.modal-fondo {
  position: fixed;

  inset: 0;

  background: rgba(0, 0, 0, .65);

  display: flex;

  align-items: center;

  justify-content: center;

  padding: 20px;

  z-index: 100;
}

.modal {
  width: 100%;

  max-width: 570px;

  max-height: 92vh;

  overflow-y: auto;

  background: white;

  border-radius: 8px;
}

.modal-header {
  display: flex;

  justify-content: space-between;

  padding: 20px;

  border-bottom: 1px solid #ddd;
}

.modal-header h2 {
  margin: 0 0 5px;

  font-size: 20px;
}

.modal-header p {
  margin: 0;

  color: #888;

  font-size: 12px;
}

.cerrar {
  width: 32px;
  height: 32px;

  border: none;

  background: #eee;

  border-radius: 50%;

  font-size: 20px;

  cursor: pointer;
}




.formulario {
  padding: 20px;

  display: flex;

  flex-direction: column;

  gap: 13px;
}

.fila {
  display: grid;

  grid-template-columns: 1fr 1fr;

  gap: 12px;
}

.campo {
  display: flex;

  flex-direction: column;

  gap: 6px;
}

.campo label {
  font-size: 11px;

  font-weight: bold;

  color: #555;
}

.campo label span {
  color: #999;

  font-weight: normal;
}

.campo input,
.campo select,
.campo textarea {
  width: 100%;

  padding: 10px;

  border: 1px solid #d5d2cd;

  border-radius: 5px;

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
  height: 75px;

  resize: vertical;
}



.mensaje-error {
  padding: 10px;

  background: #f9e6e6;

  border: 1px solid #eccaca;

  color: #a04d4d;

  border-radius: 5px;

  font-size: 12px;
}




.botones-formulario {
  display: grid;

  grid-template-columns: 1fr 1fr;

  gap: 10px;

  margin-top: 5px;
}

.boton-cancelar,
.boton-guardar {
  padding: 11px;

  border-radius: 5px;

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


</style>
