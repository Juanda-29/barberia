<script setup>
import { ref } from 'vue'
import { useLocalStorage } from '@vueuse/core'

const mostrarModal = ref(false)

const servicios = useLocalStorage('servicios-barberia', [])

const cliente = ref('')
const servicio = ref('')
const barbero = ref('')
const fecha = ref('')
const precio = ref('')
const metodo = ref('')
const estado = ref('')

function abrirModal() {
  mostrarModal.value = true
}

function cerrarModal() {
  mostrarModal.value = false
}

function guardarServicio() {

  if (cliente.value == '' || servicio.value == '') {
    alert('Complete los campos principales')
    return
  }

  servicios.value.push({
    id: Date.now(),
    cliente: cliente.value,
    servicio: servicio.value,
    barbero: barbero.value,
    fecha: fecha.value,
    precio: precio.value,
    metodo: metodo.value,
    estado: estado.value
  })

  cliente.value = ''
  servicio.value = ''
  barbero.value = ''
  fecha.value = ''
  precio.value = ''
  metodo.value = ''
  estado.value = ''

  cerrarModal()
}
</script>


<template>

  <div class="pagina">

    <!-- ENCABEZADO -->

    <header class="header">

      <div class="titulo">

        <h1>
          ✂ Barbería Don Ramiro
        </h1>

        <p>
          Registro de servicios
        </p>

      </div>

      <button
        class="btn-nuevo"
        @click="abrirModal"
      >
        + Nuevo servicio
      </button>

    </header>


    <!-- CONTENIDO -->

    <main class="contenido">

      <h2>
        Servicios registrados
      </h2>

      <p class="texto">
        Aquí se muestran los servicios realizados en la barbería.
      </p>


      <!-- CUANDO NO HAY SERVICIOS -->

      <div
        v-if="servicios.length == 0"
        class="mensaje"
      >

        <div class="icono">
          ✂
        </div>

        <h3>
          No hay servicios
        </h3>

        <p>
          Registre el primer servicio de la barbería.
        </p>

        <button
          class="btn-nuevo"
          @click="abrirModal"
        >
          Registrar servicio
        </button>

      </div>


      <!-- TARJETAS -->

      <div
        v-if="servicios.length > 0"
        class="tarjetas"
      >

        <div
          v-for="item in servicios"
          :key="item.id"
          class="tarjeta"
        >

          <h3>
            {{ item.cliente }}
          </h3>

          <p>
            <b>Servicio:</b>
            {{ item.servicio }}
          </p>

          <p>
            <b>Barbero:</b>
            {{ item.barbero }}
          </p>

          <p>
            <b>Fecha:</b>
            {{ item.fecha }}
          </p>

          <p>
            <b>Precio:</b>
            ${{ item.precio }}
          </p>

          <p v-if="item.metodo">
            <b>Pago:</b>
            {{ item.metodo }}
          </p>

          <span
            v-if="item.estado"
            class="estado"
          >
            {{ item.estado }}
          </span>

        </div>

      </div>

    </main>


    <!-- MODAL -->

    <div
      v-if="mostrarModal"
      class="modal-fondo"
    >

      <div class="modal">

        <div class="modal-titulo">

          <h2>
            Nuevo servicio
          </h2>

          <button
            class="cerrar"
            @click="cerrarModal"
          >
            ×
          </button>

        </div>


        <form @submit.prevent="guardarServicio">

          <!-- CLIENTE -->

          <label>
            Nombre del cliente
          </label>

          <input
            v-model="cliente"
            type="text"
            placeholder="Nombre"
          >


          <!-- SERVICIO -->

          <label>
            Tipo de servicio
          </label>

          <select v-model="servicio">

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


          <!-- BARBERO -->

          <label>
            Barbero
          </label>

          <select v-model="barbero">

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


          <!-- FECHA -->

          <label>
            Fecha
          </label>

          <input
            v-model="fecha"
            type="date"
          >


          <!-- PRECIO -->

          <label>
            Precio
          </label>

          <input
            v-model="precio"
            type="number"
            placeholder="20000"
          >


          <!-- METODO -->

          <label>
            Método de pago
          </label>

          <select v-model="metodo">

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


          <!-- ESTADO -->

          <label>
            Estado del pago
          </label>

          <select v-model="estado">

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


          <!-- BOTONES -->

          <div class="botones">

            <button
              type="button"
              class="btn-cancelar"
              @click="cerrarModal"
            >
              Cancelar
            </button>

            <button
              type="submit"
              class="btn-guardar"
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
  background: #f4f4f4;
  font-family: Arial, sans-serif;
  color: #333;
}


/* HEADER */

.header {
  background: #222;
  color: white;

  padding: 20px 7%;

  display: flex;
  justify-content: space-between;
  align-items: center;
}

.titulo h1 {
  margin: 0;
  font-size: 25px;
}

.titulo p {
  margin: 5px 0 0;
  color: #bbb;
  font-size: 14px;
}

.btn-nuevo {
  border: none;
  background: #c59b5c;
  color: #222;

  padding: 10px 15px;

  border-radius: 5px;

  cursor: pointer;

  font-weight: bold;
}

.btn-nuevo:hover {
  background: #d6ad70;
}


/* CONTENIDO */

.contenido {
  width: 90%;
  max-width: 1000px;

  margin: 30px auto;
}

.contenido h2 {
  margin-bottom: 5px;
}

.texto {
  color: #777;
  font-size: 14px;
}


/* MENSAJE */

.mensaje {
  margin-top: 30px;

  background: white;

  border: 1px solid #ddd;

  padding: 50px 20px;

  text-align: center;

  border-radius: 7px;
}

.icono {
  font-size: 35px;
}

.mensaje p {
  color: #777;
}


/* TARJETAS */

.tarjetas {
  margin-top: 25px;

  display: grid;

  grid-template-columns: repeat(3, 1fr);

  gap: 15px;
}

.tarjeta {
  background: white;

  border: 1px solid #ddd;

  border-radius: 7px;

  padding: 18px;
}

.tarjeta h3 {
  margin-top: 0;

  border-bottom: 1px solid #eee;

  padding-bottom: 10px;
}

.tarjeta p {
  font-size: 13px;

  margin: 9px 0;
}

.estado {
  display: inline-block;

  margin-top: 5px;

  padding: 5px 8px;

  background: #eee;

  border-radius: 4px;

  font-size: 11px;
}


/* MODAL */

.modal-fondo {
  position: fixed;

  top: 0;
  left: 0;

  width: 100%;
  height: 100%;

  background: rgba(0, 0, 0, .6);

  display: flex;

  justify-content: center;

  align-items: center;

  padding: 20px;
}

.modal {
  background: white;

  width: 100%;

  max-width: 500px;

  max-height: 90vh;

  overflow-y: auto;

  border-radius: 7px;

  padding: 20px;
}

.modal-titulo {
  display: flex;

  justify-content: space-between;

  align-items: center;

  margin-bottom: 15px;
}

.modal-titulo h2 {
  margin: 0;
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
  display: flex;

  flex-direction: column;

  gap: 7px;
}

label {
  font-size: 13px;

  font-weight: bold;

  margin-top: 5px;
}

input,
select {
  padding: 10px;

  border: 1px solid #ccc;

  border-radius: 4px;

  font-size: 13px;
}

input:focus,
select:focus {
  outline: none;

  border-color: #c59b5c;
}


/* BOTONES */

.botones {
  display: flex;

  gap: 10px;

  margin-top: 15px;
}

.btn-cancelar,
.btn-guardar {
  flex: 1;

  padding: 10px;

  border-radius: 5px;

  cursor: pointer;

  font-weight: bold;
}

.btn-cancelar {
  border: 1px solid #ccc;

  background: white;
}

.btn-guardar {
  border: none;

  background: #222;

  color: white;
}


/* RESPONSIVE */

@media (max-width: 700px) {

  .header {
    flex-direction: column;

    align-items: flex-start;

    gap: 15px;
  }

  .btn-nuevo {
    width: 100%;
  }

  .tarjetas {
    grid-template-columns: 1fr;
  }

}

</style>
