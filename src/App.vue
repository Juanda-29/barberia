<script setup>
import { ref } from 'vue'
import { useLocalStorage } from '@vueuse/core'

const tiposServicio = [
  'Corte clásico',
  'Corte moderno',
  'Barba',
  'Cejas',
  'Tinte'
]

const barberos = [
  'Don Ramiro',
  'Pedrito',
  'Juanito'
]

const metodosPago = [
  'Efectivo',
  'Transferencia',
  'Tarjeta'
]

const estadosPago = [
  'Pagado',
  'Pendiente',
  'Abonado'
]

const preciosServicios = {
  'Corte clásico': 25000,
  'Corte moderno': 18000,
  'Barba': 10000,
  'Cejas': 8000,
  'Tinte': 30000
}

const servicios = useLocalStorage('servicios-barberia', [])

const mostrarModal = ref(false)
const editando = ref(false)
const idEditando = ref(null)
const errorFormulario = ref('')

const idParaEliminar = ref(null)

function formularioVacio() {
  return {
    cliente: '',
    tipos: [],
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

// suma el precio de todos los servicios marcados
function actualizarPrecio() {
  let total = 0

  form.value.tipos.forEach(tipo => {
    total += preciosServicios[tipo] || 0
  })

  form.value.precio = total
}

function abrirModalNuevo() {
  form.value = formularioVacio()
  editando.value = false
  idEditando.value = null
  errorFormulario.value = ''
  mostrarModal.value = true
}

function editarServicio(servicio) {
  
  let tiposCargados = []

  if (Array.isArray(servicio.tipos)) {
    tiposCargados = [...servicio.tipos]
  } else if (servicio.tipoServicio) {
    tiposCargados = [servicio.tipoServicio]
  }

  form.value = {
    cliente: servicio.cliente,
    tipos: tiposCargados,
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
  if (form.value.tipos.length === 0) {
    return 'Seleccione al menos un servicio.'
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
    return 'El precio debe ser mayor a 0.'
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
    const posicion = servicios.value.findIndex(s => s.id === idEditando.value)

    if (posicion !== -1) {
      servicios.value[posicion] = {
        id: idEditando.value,
        cliente: form.value.cliente,
        tipos: [...form.value.tipos],
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
      tipos: [...form.value.tipos],
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

function pedirConfirmacionEliminar(id) {
  idParaEliminar.value = id
}

function cancelarEliminar() {
  idParaEliminar.value = null
}

function confirmarEliminar() {
  servicios.value = servicios.value.filter(s => s.id !== idParaEliminar.value)
  idParaEliminar.value = null
}

function nombreServicioAEliminar() {
  const servicio = servicios.value.find(s => s.id === idParaEliminar.value)
  return servicio ? servicio.cliente : ''
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
  return servicios.value.filter(s => s.estadoPago === 'Pagado').length
}

function contarPendientes() {
  return servicios.value.filter(s => s.estadoPago === 'Pendiente').length
}

function contarAbonados() {
  return servicios.value.filter(s => s.estadoPago === 'Abonado').length
}

function totalVentas() {
  let total = 0
  servicios.value.forEach(s => {
    if (s.estadoPago === 'Pagado') {
      total += Number(s.precio)
    }
  })
  return total
}

function generarEstrellas(calificacion) {
  let estrellas = ''
  for (let i = 1; i <= 5; i++) {
    estrellas += i <= Number(calificacion) ? '★' : '☆'
  }
  return estrellas
}


function listaServicios(servicio) {
  if (Array.isArray(servicio.tipos) && servicio.tipos.length > 0) {
    return servicio.tipos.join(', ')
  }
  if (servicio.tipoServicio) {
    return servicio.tipoServicio
  }
  return 'Sin servicio'
}
</script>

<template>
  <div class="pagina">

    <header class="header">
      <div class="header-interior">
        <div>
          <h1>BARBERIA DON RAMIRO</h1>
          <p>CONTROL DE SERVICIOS</p>
        </div>

        <button class="boton-nuevo" @click="abrirModalNuevo">
          + NUEVO SERVICIO
        </button>
      </div>
    </header>

    <main class="contenedor">

      <section class="resumen">
        <div class="resumen-item">
          <span class="numero">{{ totalServicios() }}</span>
          <span class="texto">SERVICIOS</span>
        </div>
        <div class="resumen-item">
          <span class="numero">{{ contarPagados() }}</span>
          <span class="texto">PAGADOS</span>
        </div>
        <div class="resumen-item">
          <span class="numero">{{ contarPendientes() }}</span>
          <span class="texto">PENDIENTES</span>
        </div>
        <div class="resumen-item">
          <span class="numero">{{ contarAbonados() }}</span>
          <span class="texto">ABONADOS</span>
        </div>
        <div class="resumen-item">
          <span class="numero">${{ totalVentas().toLocaleString('es-CO') }}</span>
          <span class="texto">VENTAS PAGADAS</span>
        </div>
      </section>

      <h2 class="titulo-seccion">Servicios registrados</h2>

      <section v-if="servicios.length === 0" class="vacio">
        <p>No hay servicios registrados todavía.</p>
        <button class="boton-nuevo" @click="abrirModalNuevo">
          + NUEVO SERVICIO
        </button>
      </section>

      <section v-if="servicios.length > 0" class="lista">
        <article v-for="servicio in servicios" :key="servicio.id" class="tarjeta">

          <div class="tarjeta-top">
            <h3>{{ servicio.cliente }}</h3>
            <strong class="precio">${{ Number(servicio.precio || 0).toLocaleString('es-CO') }}</strong>
          </div>

          <p class="linea">{{ listaServicios(servicio) }}</p>
          <p class="linea">Atendido por {{ servicio.barbero }}</p>
          <p class="linea">{{ servicio.fecha }} · {{ servicio.hora }}</p>
          <p class="linea">Pago: {{ servicio.metodoPago }}</p>

          <p class="linea">
            <span v-if="servicio.estadoPago === 'Pagado'" class="estado ok">Pagado</span>
            <span v-else-if="servicio.estadoPago === 'Pendiente'" class="estado pendiente">Pendiente</span>
            <span v-else class="estado abonado">Abonado</span>

            <span class="estrellas">{{ generarEstrellas(servicio.calificacion) }}</span>
          </p>

          <p v-if="servicio.observaciones" class="observacion">
            {{ servicio.observaciones }}
          </p>

          <div class="acciones">
            <button class="boton-editar" @click="editarServicio(servicio)">Editar</button>
            <button class="boton-eliminar" @click="pedirConfirmacionEliminar(servicio.id)">Eliminar</button>
          </div>

        </article>
      </section>

    </main>

    <div v-if="mostrarModal" class="modal-fondo">
      <div class="modal">

        <div class="modal-header">
          <h2>{{ editando ? 'Editar servicio' : 'Nuevo servicio' }}</h2>
          <button class="cerrar" @click="cancelar">×</button>
        </div>

        <form class="formulario" @submit.prevent="guardarServicio">

          <div class="campo">
            <label>Nombre del cliente</label>
            <input v-model="form.cliente" type="text" />
          </div>

          <div class="campo">
            <label>Servicios</label>

            <label v-for="tipo in tiposServicio" :key="tipo" class="opcion-check">
              <input
                type="checkbox"
                :value="tipo"
                v-model="form.tipos"
                @change="actualizarPrecio"
              />
              {{ tipo }} — ${{ preciosServicios[tipo].toLocaleString('es-CO') }}
            </label>

            <p class="total-preview">
              Total: ${{ Number(form.precio || 0).toLocaleString('es-CO') }}
            </p>
          </div>

          <div class="campo">
            <label>Barbero</label>
            <select v-model="form.barbero">
              <option value="" disabled>Seleccione</option>
              <option v-for="barbero in barberos" :key="barbero" :value="barbero">
                {{ barbero }}
              </option>
            </select>
          </div>

          <div class="fila">
            <div class="campo">
              <label>Fecha</label>
              <input v-model="form.fecha" type="date" />
            </div>
            <div class="campo">
              <label>Hora</label>
              <input v-model="form.hora" type="time" />
            </div>
          </div>

          <div class="campo">
            <label>Precio total</label>
            <input v-model="form.precio" type="number" min="1" />
          </div>

          <div class="fila">
            <div class="campo">
              <label>Método de pago</label>
              <select v-model="form.metodoPago">
                <option value="" disabled>Seleccione</option>
                <option v-for="metodo in metodosPago" :key="metodo" :value="metodo">
                  {{ metodo }}
                </option>
              </select>
            </div>

            <div class="campo">
              <label>Estado del pago</label>
              <select v-model="form.estadoPago">
                <option value="" disabled>Seleccione</option>
                <option v-for="estado in estadosPago" :key="estado" :value="estado">
                  {{ estado }}
                </option>
              </select>
            </div>
          </div>

          <div class="campo">
            <label>Calificación</label>
            <select v-model="form.calificacion">
              <option value="" disabled>Seleccione</option>
              <option value="1">★☆☆☆☆</option>
              <option value="2">★★☆☆☆</option>
              <option value="3">★★★☆☆</option>
              <option value="4">★★★★☆</option>
              <option value="5">★★★★★</option>
            </select>
          </div>

          <div class="campo">
            <label>Observaciones (opcional)</label>
            <textarea v-model="form.observaciones"></textarea>
          </div>

          <div v-show="errorFormulario" class="mensaje-error">
            {{ errorFormulario }}
          </div>

          <div class="botones-formulario">
            <button type="button" class="boton-cancelar" @click="cancelar">Cancelar</button>
            <button type="submit" class="boton-guardar">
              {{ editando ? 'Guardar cambios' : 'Guardar servicio' }}
            </button>
          </div>

        </form>
      </div>
    </div>

    <div v-if="idParaEliminar !== null" class="confirmar-fondo">
      <div class="confirmar-caja">
        <h3>Eliminar servicio</h3>
        <p>¿Estás seguro que quieres eliminar el servicio de <strong>{{ nombreServicioAEliminar() }}</strong>?</p>

        <div class="confirmar-botones">
          <button class="boton-cancelar" @click="cancelarEliminar">Cancelar</button>
          <button class="boton-eliminar-confirmar" @click="confirmarEliminar">Sí, eliminar</button>
        </div>
      </div>
    </div>

  </div>
</template>

<style scoped>

* {
  box-sizing: border-box;
}

.pagina {
  min-height: 100vh;
  background: #fffffe;
  color: #222;
  font-family: Arial, Helvetica, sans-serif;
}

.header {
  background: #88664a;
  color: #fff;
}

.header-interior {
  max-width: 1000px;
  margin: 0 auto;
  padding: 24px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 16px;
  flex-wrap: wrap;
}

.header h1 {
  margin: 0;
  font-size: 22px;
}

.header p {
  margin: 4px 0 0;
  color: #bbb;
  font-size: 13px;
}

.boton-nuevo {
  border: 1px solid #333;
  background: #333;
  color: #fff;
  padding: 10px 16px;
  border-radius: 4px;
  font-size: 14px;
  cursor: pointer;
}

.contenedor {
  width: 92%;
  max-width: 1000px;
  margin: 24px auto 50px;
}

.resumen {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
  margin-bottom: 24px;
}

.resumen-item {
  background: #c9e7e9;
  border: 1px solid #ddd;
  border-radius: 4px;
  padding: 12px 16px;
  flex: 1;
  min-width: 110px;
  text-align: center;
}

.resumen-item .numero {
  display: block;
  font-size: 18px;
  font-weight: bold;
}

.resumen-item .texto {
  font-size: 12px;
  color: #666;
}

.titulo-seccion {
  font-size: 18px;
  margin-bottom: 12px;
}

.vacio {
  background: #fff;
  border: 1px solid #ddd;
  border-radius: 4px;
  padding: 30px;
  text-align: center;
}

.lista {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 14px;
}

.tarjeta {
  background: #fff;
  border: 1px solid #ddd;
  border-radius: 4px;
  padding: 14px;
}

.tarjeta-top {
  display: flex;
  justify-content: space-between;
  align-items: baseline;
  margin-bottom: 6px;
}

.tarjeta-top h3 {
  margin: 0;
  font-size: 15px;
}

.precio {
  font-size: 14px;
}

.linea {
  margin: 4px 0;
  font-size: 13px;
  color: #444;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.estado {
  font-size: 12px;
  font-weight: bold;
  padding: 2px 6px;
  border-radius: 3px;
}

.estado.ok {
  background: #e4f2e6;
  color: #2f6b3a;
}

.estado.pendiente {
  background: #fdf1dc;
  color: #8a5a13;
}

.estado.abonado {
  background: #f7ddcc;
  color: #d17812;
}

.estrellas {
  color: #b8860b;
  font-size: 13px;
}

.observacion {
  margin-top: 8px;
  font-size: 12px;
  color: #555;
  background: #f7f7f7;
  padding: 6px 8px;
  border-radius: 4px;
}

.acciones {
  display: flex;
  gap: 8px;
  margin-top: 10px;
}

.boton-editar,
.boton-eliminar {
  flex: 1;
  padding: 7px;
  border-radius: 4px;
  font-size: 12px;
  cursor: pointer;
}

.boton-editar {
  border: 1px solid #ccc;
  background: #fff;
}

.boton-eliminar {
  border: 1px solid #e3b8b8;
  background: #fff;
  color: #a03a3a;
}

.modal-fondo {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 16px;
  z-index: 100;
}

.modal {
  width: 100%;
  max-width: 460px;
  max-height: 90vh;
  overflow-y: auto;
  background: #fff;
  border-radius: 4px;
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 14px 16px;
  border-bottom: 1px solid #eee;
}

.modal-header h2 {
  margin: 0;
  font-size: 15px;
}

.cerrar {
  border: none;
  background: none;
  font-size: 18px;
  cursor: pointer;
}

.formulario {
  padding: 16px;
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.fila {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 10px;
}

.campo {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.campo label {
  font-size: 12px;
  font-weight: bold;
  color: #444;
}

.campo input,
.campo select,
.campo textarea {
  border: none;
  border-bottom: 1px solid #ccc;
  padding: 6px 2px;
  font-size: 13px;
  background: transparent;
}

.campo input:focus,
.campo select:focus,
.campo textarea:focus {
  outline: none;
  border-bottom-color: #333;
}

.campo textarea {
  height: 55px;
  resize: vertical;
}

.opcion-check {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 13px;
  font-weight: normal;
  color: #333;
  padding: 3px 0;
}

.opcion-check input {
  width: auto;
}

.total-preview {
  margin: 4px 0 0;
  font-size: 13px;
  font-weight: bold;
}

.mensaje-error {
  background: #fbe6e6;
  color: #8c3535;
  padding: 8px;
  border-radius: 4px;
  font-size: 12px;
}

.botones-formulario {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 10px;
  margin-top: 4px;
}

.boton-cancelar,
.boton-guardar {
  padding: 9px;
  border-radius: 4px;
  cursor: pointer;
  font-weight: bold;
}

.boton-cancelar {
  border: 1px solid #ccc;
  background: #fff;
}

.boton-guardar {
  border: none;
  background: #1f1f1f;
  color: #fff;
}

.confirmar-fondo {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 16px;
  z-index: 200;
}

.confirmar-caja {
  background: #fff;
  width: 100%;
  max-width: 340px;
  border-radius: 4px;
  padding: 18px;
}

.confirmar-caja h3 {
  margin: 0 0 8px;
  font-size: 15px;
}

.confirmar-caja p {
  margin: 0 0 16px;
  font-size: 13px;
  color: #444;
}

.confirmar-botones {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 10px;
}

.boton-eliminar-confirmar {
  border: none;
  background: #a03a3a;
  color: #fff;
  padding: 9px;
  border-radius: 4px;
  cursor: pointer;
  font-weight: bold;
}

</style>
