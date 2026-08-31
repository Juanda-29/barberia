```vue
<script setup>
import { ref } from 'vue'
import { useLocalStorage } from '@vueuse/core'

const mostrarModal = ref(false)
const editando = ref(false)
const idEditar = ref(null)

const servicios = useLocalStorage('servicios', [])

const formulario = ref({
  cliente: '',
  servicio: '',
  barbero: '',
  fecha: '',
  hora: '',
  precio: '',
  metodo: '',
  estado: '',
  calificacion: '',
  observaciones: ''
})

function abrirModal() {
  mostrarModal.value = true
  editando.value = false
  limpiarFormulario()
}

function cerrarModal() {
  mostrarModal.value = false
  limpiarFormulario()
}

function limpiarFormulario() {
  formulario.value = {
    cliente: '',
    servicio: '',
    barbero: '',
    fecha: '',
    hora: '',
    precio: '',
    metodo: '',
    estado: '',
    calificacion: '',
    observaciones: ''
  }
}

function guardarServicio() {

  if (
    !formulario.value.cliente ||
    !formulario.value.servicio ||
    !formulario.value.barbero ||
    !formulario.value.fecha ||
    !formulario.value.precio
  ) {
    alert('Complete los campos obligatorios')
    return
  }

  if (editando.value) {

    for (let i = 0; i < servicios.value.length; i++) {

      if (servicios.value[i].id === idEditar.value) {
        servicios.value[i] = {
          id: idEditar.value,
          ...formulario.value
        }
      }

    }

  } else {

    servicios.value.push({
      id: Date.now(),
      ...formulario.value
    })

  }

  cerrarModal()
}

function editarServicio(servicio) {

  formulario.value = {
    cliente: servicio.cliente,
    servicio: servicio.servicio,
    barbero: servicio.barbero,
    fecha: servicio.fecha,
    hora: servicio.hora,
    precio: servicio.precio,
    metodo: servicio.metodo,
    estado: servicio.estado,
    calificacion: servicio.calificacion,
    observaciones: servicio.observaciones
  }

  idEditar.value = servicio.id
  editando.value = true
  mostrarModal.value = true
}

function eliminarServicio(id) {

  const confirmar = confirm(
    '¿Está seguro de eliminar este servicio?'
  )

  if (confirmar) {

    const nuevosServicios = []

    for (let i = 0; i < servicios.value.length; i++) {

      if (servicios.value[i].id !== id) {
        nuevosServicios.push(servicios.value[i])
      }

    }

    servicios.value = nuevosServicios
  }
}
</script>


<template>

  <div class="pagina">

    <header class="encabezado">

      <div>
        <h1>✂ Barbería Don Ramiro</h1>
        <p>Registro de servicios</p>
      </div>

      <button
        class="boton-principal"
        @click="abrirModal"
      >
        + Nuevo servicio
      </button>

    </header>


    <main class="contenido">

      <div class="titulo">

        <h2>Servicios</h2>

        <p>
          Registre los servicios realizados en la barbería.
        </p>

      </div>


      <div
        v-if="servicios.length === 0"
        class="mensaje"
      >

        <h3>No hay servicios registrados</h3>

        <p>
          Presione el botón para registrar el primer servicio.
        </p>

      </div>


      <div
        v-if="servicios.length > 0"
        class="tarjetas"
      >

        <div
          v-for="servicio in servicios"
          :key="servicio.id"
          class="tarjeta"
        >

          <div class="tarjeta-arriba">

            <div>

              <h3>
                {{ servicio.cliente }}
              </h3>

              <p>
                {{ servicio.servicio }}
              </p>

            </div>

            <strong>
              ${{ servicio.precio }}
            </strong>

          </div>


          <div class="informacion">

            <p>
              <b>Barbero:</b>
              {{ servicio.barbero }}
            </p>

            <p>
              <b>Fecha:</b>
              {{ servicio.fecha }}
            </p>

            <p>
              <b>Hora:</b>
              {{ servicio.hora }}
            </p>

            <p v-if="servicio.metodo">
              <b>Pago:</b>
              {{ servicio.metodo }}
            </p>

            <p
              v-if="servicio.estado"
              :class="{
                pendiente: servicio.estado === 'Pendiente',
                fiado: servicio.estado === 'Fiado'
              }"
            >
              <b>Estado:</b>
              {{ servicio.estado }}
            </p>

            <p v-if="servicio.calificacion">
              <b>Calificación:</b>
              {{ servicio.calificacion }} / 5
            </p>

            <p v-if="servicio.observaciones">
              <b>Observación:</b>
              {{ servicio.observaciones }}
            </p>

          </div>


          <div class="acciones">

            <button
              class="editar"
              @click="editarServicio(servicio)"
            >
              Editar
            </button>

            <button
              class="eliminar"
              @click="eliminarServicio(servicio.id)"
            >
              Eliminar
            </button>

          </div>

        </div>

      </div>

    </main>


    <!-- MODAL -->

    <div
      v-if="mostrarModal"
      class="fondo-modal"
    >

      <div class="modal">

        <div class="modal-titulo">

          <h2 v-if="!editando">
            Nuevo servicio
          </h2>

          <h2 v-else>
            Editar servicio
          </h2>

          <button
            class="cerrar"
            @click="cerrarModal"
          >
            ×
          </button>

        </div>


        <form
          @submit.prevent="guardarServicio"
        >

          <div class="campo">

            <label>Nombre del cliente *</label>

            <input
              v-model="formulario.cliente"
              type="text"
              placeholder="Nombre del cliente"
            >

          </div>


          <div class="fila">

            <div class="campo">

              <label>Servicio *</label>

              <select v-model="formulario.servicio">

                <option value="">
                  Seleccione
                </option>

                <option>
                  Corte clásico
                </option>

                <option>
                  Corte moderno
                </option>

                <option>
                  Barba
                </option>

                <option>
                  Corte + barba
                </option>

                <option>
                  Cejas
                </option>

                <option>
                  Tinte
                </option>

              </select>

            </div>


            <div class="campo">

              <label>Barbero *</label>

              <select v-model="formulario.barbero">

                <option value="">
                  Seleccione
                </option>

                <option>
                  Don Ramiro
                </option>

                <option>
                  Carlos
                </option>

                <option>
                  Andrés
                </option>

              </select>

            </div>

          </div>


          <div class="fila">

            <div class="campo">

              <label>Fecha *</label>

              <input
                v-model="formulario.fecha"
                type="date"
              >

            </div>


            <div class="campo">

              <label>Hora</label>

              <input
                v-model="formulario.hora"
                type="time"
              >

            </div>

          </div>


          <div class="campo">

            <label>Precio *</label>

            <input
              v-model="formulario.precio"
              type="number"
              placeholder="20000"
            >

          </div>


          <div class="fila">

            <div class="campo">

              <label>Método de pago</label>

              <select v-model="formulario.metodo">

                <option value="">
                  Seleccione
                </option>

                <option>
                  Efectivo
                </option>

                <option>
                  Transferencia
                </option>

                <option>
                  Tarjeta
                </option>

              </select>

            </div>


            <div class="campo">

              <label>Estado del pago</label>

              <select v-model="formulario.estado">

                <option value="">
                  Seleccione
                </option>

                <option>
                  Pagado
                </option>

                <option>
                  Pendiente
                </option>

                <option>
                  Fiado
                </option>

              </select>

            </div>

          </div>


          <div class="campo">

            <label>Calificación</label>

            <select v-model="formulario.calificacion">

              <option value="">
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

            <label>Observaciones</label>

            <textarea
              v-model="formulario.observaciones"
              placeholder="Observaciones del servicio"
            ></textarea>

          </div>


          <div class="botones">

            <button
              type="button"
              class="cancelar"
              @click="cerrarModal"
            >
              Cancelar
            </button>

            <button
              type="submit"
              class="guardar"
            >
              Guardar
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
  background: #f4f2ef;
  color: #292929;
  font-family: Arial, sans-serif;
}


/* ENCABEZADO */

.encabezado {
  background: #292929;
  color: white;

  padding: 25px 7%;

  display: flex;
  align-items: center;
  justify-content: space-between;
}

.encabezado h1 {
  margin: 0;
  font-size: 25px;
}

.encabezado p {
  margin: 7px 0 0;
  color: #bbb;
}


/* BOTON */

.boton-principal {
  background: #c59b59;
  color: #222;

  border: none;
  border-radius: 5px;

  padding: 11px 16px;

  cursor: pointer;
  font-weight: bold;
}

.boton-principal:hover {
  background: #d3ac6b;
}


/* CONTENIDO */

.contenido {
  width: 86%;
  max-width: 1100px;

  margin: 35px auto;
}

.titulo {
  margin-bottom: 25px;
}

.titulo h2 {
  margin-bottom: 5px;
}

.titulo p {
  color: #777;
  font-size: 14px;
}


/* MENSAJE */

.mensaje {
  background: white;

  border: 1px solid #ddd;

  border-radius: 6px;

  padding: 45px;

  text-align: center;
}

.mensaje h3 {
  margin-bottom: 5px;
}

.mensaje p {
  color: #777;
}


/* TARJETAS */

.tarjetas {
  display: grid;

  grid-template-columns: repeat(3, 1fr);

  gap: 18px;
}

.tarjeta {
  background: white;

  border: 1px solid #ddd;

  border-radius: 6px;

  padding: 18px;
}

.tarjeta-arriba {
  display: flex;

  justify-content: space-between;

  border-bottom: 1px solid #eee;

  padding-bottom: 12px;
}

.tarjeta-arriba h3 {
  margin: 0;
  font-size: 17px;
}

.tarjeta-arriba p {
  margin: 5px 0;

  color: #777;

  font-size: 12px;
}

.tarjeta-arriba strong {
  color: #9a7135;
}

.informacion {
  padding: 12px 0;
}

.informacion p {
  margin: 8px 0;

  font-size: 13px;
}

.informacion b {
  color: #555;
}

.pendiente {
  color: #a36d22;
}

.fiado {
  color: #a33d3d;
}


/* ACCIONES */

.acciones {
  display: flex;
  gap: 8px;
}

.acciones button {
  flex: 1;

  padding: 8px;

  border-radius: 4px;

  cursor: pointer;
}

.editar {
  background: white;
  border: 1px solid #bbb;
}

.eliminar {
  background: white;
  border: 1px solid #d1a5a5;

  color: #9b4545;
}


/* MODAL */

.fondo-modal {
  position: fixed;

  top: 0;
  left: 0;
  right: 0;
  bottom: 0;

  background: rgba(0, 0, 0, .6);

  display: flex;

  align-items: center;
  justify-content: center;

  padding: 20px;
}

.modal {
  background: white;

  width: 100%;
  max-width: 600px;

  max-height: 90vh;

  overflow-y: auto;

  border-radius: 7px;
}

.modal-titulo {
  padding: 18px 20px;

  border-bottom: 1px solid #ddd;

  display: flex;

  align-items: center;

  justify-content: space-between;
}

.modal-titulo h2 {
  margin: 0;
  font-size: 20px;
}

.cerrar {
  border: none;

  background: #eee;

  width: 30px;
  height: 30px;

  border-radius: 50%;

  font-size: 20px;

  cursor: pointer;
}


/* FORMULARIO */

form {
  padding: 20px;
}

.campo {
  margin-bottom: 14px;
}

.campo label {
  display: block;

  margin-bottom: 5px;

  font-size: 12px;

  font-weight: bold;
}

.campo input,
.campo select,
.campo textarea {
  width: 100%;

  padding: 9px;

  border: 1px solid #ccc;

  border-radius: 4px;

  font-size: 13px;
}

.campo textarea {
  height: 70px;

  resize: vertical;
}

.fila {
  display: grid;

  grid-template-columns: 1fr 1fr;

  gap: 12px;
}


/* BOTONES DEL FORMULARIO */

.botones {
  display: flex;

  justify-content: flex-end;

  gap: 10px;

  margin-top: 20px;
}

.cancelar,
.guardar {
  padding: 10px 18px;

  border-radius: 4px;

  cursor: pointer;

  font-weight: bold;
}

.cancelar {
  background: white;

  border: 1px solid #bbb;
}

.guardar {
  background: #292929;

  color: white;

  border: none;
}


/* CELULAR */

@media (max-width: 750px) {

  .encabezado {
    padding: 20px;

    flex-direction: column;

    align-items: flex-start;

    gap: 15px;
  }

  .boton-principal {
    width: 100%;
  }

  .contenido {
    width: 92%;
  }

  .tarjetas {
    grid-template-columns: 1fr;
  }

  .fila {
    grid-template-columns: 1fr;
  }

}

</style>
