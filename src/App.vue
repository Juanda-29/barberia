```vue
<script setup>
import { ref } from 'vue'
import { useLocalStorage } from '@vueuse/core'

const mostrarModal = ref(false)

const servicios = useLocalStorage('servicios-barberia', [])

const cliente = ref('')
const servicio = ref('')
const barbero = ref('')
const fecha = ref('')
const hora = ref('')
const precio = ref('')
const metodo = ref('')
const estado = ref('')
const calificacion = ref('')
const observaciones = ref('')

function abrirModal() {
  mostrarModal.value = true
}

function cerrarModal() {
  mostrarModal.value = false
}

function guardarServicio() {

  if (cliente.value == '' || servicio.value == '' || precio.value == '') {
    alert('Por favor complete los campos principales')
    return
  }

  servicios.value.push({
    id: Date.now(),
    cliente: cliente.value,
    servicio: servicio.value,
    barbero: barbero.value,
    fecha: fecha.value,
    hora: hora.value,
    precio: precio.value,
    metodo: metodo.value,
    estado: estado.value,
    calificacion: calificacion.value,
    observaciones: observaciones.value
  })

  cliente.value = ''
  servicio.value = ''
  barbero.value = ''
  fecha.value = ''
  hora.value = ''
  precio.value = ''
  metodo.value = ''
  estado.value = ''
  calificacion.value = ''
  observaciones.value = ''

  cerrarModal()
}
</script>


<template>

  <div class="pagina">

    <!-- ENCABEZADO -->

    <header class="header">

      <div>
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


      <!-- SI NO HAY SERVICIOS -->

      <div
        v-if="servicios.length == 0"
        class="mensaje"
      >

        <div class="icono">
          ✂
        </div>

        <h3>
          No hay servicios todavía
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

          <div class="tarjeta-arriba">

            <div>
              <h3>
                {{ item.cliente }}
              </h3>

              <span>
                {{ item.barbero }}
              </span>
            </div>

            <strong>
              ${{ item.precio }}
            </strong>

          </div>


          <div class="informacion">

            <p>
              <b>Servicio:</b>
              {{ item.servicio }}
            </p>

            <p>
              <b>Fecha:</b>
              {{ item.fecha }}
            </p>

            <p>
              <b>Hora:</b>
              {{ item.hora }}
            </p>

            <p v-if="item.metodo">
              <b>Pago:</b>
              {{ item.metodo }}
            </p>

            <p v-if="item.estado">
              <b>Estado:</b>
              {{ item.estado }}
            </p>

            <p v-if="item.calificacion">
              <b>Calificación:</b>
              {{ item.calificacion }} / 5
            </p>

            <p v-if="item.observaciones">
              <b>Observación:</b>
              {{ item.observaciones }}
            </p>

          </div>

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


        <form
          @submit.prevent="guardarServicio"
        >

          <label>
            Nombre del cliente
          </label>

          <input
            v-model="cliente"
            type="text"
            placeholder="Nombre del cliente"
          >


          <label>
            Tipo de servicio
          </label>

          <select v-model="servicio">

            <option value="">
              Seleccione un servicio
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


          <div class="fila">

            <div>

              <label>
                Fecha
              </label>

              <input
                v-model="fecha"
                type="date"
              >

            </div>


            <div>

              <label>
                Hora
              </label>

              <input
                v-model="hora"
                type="time"
              >

            </div>

          </div>


          <label>
            Precio
          </label>

          <input
            v-model="precio"
            type="number"
            placeholder="20000"
          >


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


          <label>
            Calificación
          </label>

          <select v-model="calificacion">

            <option value="">
              Seleccione
            </option>

            <option value="1">
              ⭐ 1
            </option>

            <option value="2">
              ⭐⭐ 2
            </option>

            <option value="3">
              ⭐⭐⭐ 3
            </option>

            <option value="4">
              ⭐⭐⭐⭐ 4
            </option>

            <option value="5">
              ⭐⭐⭐⭐⭐ 5
            </option>

          </select>


          <label>
            Observaciones
          </label>

          <textarea
            v-model="observaciones"
            placeholder="Observaciones..."
          ></textarea>


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
              Guardar servicio
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
  background: #f2f2f2;
  font-family: Arial, sans-serif;
  color: #333;
}


/* HEADER */

.header {
  background: #222;
  color: white;

  padding: 25px 7%;

  display: flex;
  justify-content: space-between;
  align-items: center;
}

.header h1 {
  margin: 0;
  font-size: 25px;
}

.header p {
  margin: 6px 0 0;
  color: #bbb;
}


/* BOTON */

.btn-nuevo {
  border: none;
  background: #c49a55;
  color: white;

  padding: 11px 17px;

  border-radius: 5px;

  cursor: pointer;

  font-weight: bold;
}

.btn-nuevo:hover {
  background: #a98245;
}


/* CONTENIDO */

.contenido {
  width: 90%;
  max-width: 1100px;

  margin: 35px auto;
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
  background: white;

  border: 1px solid #ddd;

  border-radius: 7px;

  padding: 45px 20px;

  text-align: center;

  margin-top: 25px;
}

.icono {
  font-size: 30px;
  margin-bottom: 10px;
}

.mensaje h3 {
  margin: 5px;
}

.mensaje p {
  color: #777;
}


/* TARJETAS */

.tarjetas {
  display: grid;

  grid-template-columns: repeat(3, 1fr);

  gap: 18px;

  margin-top: 25px;
}

.tarjeta {
  background: white;

  border: 1px solid #ddd;

  border-radius: 7px;

  padding: 18px;
}

.tarjeta-arriba {
  display: flex;

  justify-content: space-between;

  border-bottom: 1px solid #eee;

  padding-bottom: 12px;
}

.tarjeta-arriba h3 {
  margin: 0 0 5px;
}

.tarjeta-arriba span {
  color: #888;
  font-size: 12px;
}

.tarjeta-arriba strong {
  color: #a47b39;
}

.informacion {
  margin-top: 12px;
}

.informacion p {
  font-size: 13px;
  margin: 9px 0;
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

  align-items: center;
  justify-content: center;

  padding: 20px;
}

.modal {
  background: white;

  width: 100%;
  max-width: 550px;

  max-height: 90vh;

  overflow-y: auto;

  padding: 22px;

  border-radius: 8px;
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

form label {
  display: block;

  font-size: 13px;

  font-weight: bold;

  margin: 12px 0 5px;
}

input,
select,
textarea {
  width: 100%;

  padding: 10px;

  border: 1px solid #ccc;

  border-radius: 5px;

  font-size: 13px;
}

textarea {
  height: 70px;

  resize: none;
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

.btn-cancelar {
  border: 1px solid #ccc;

  background: white;

  padding: 10px 15px;

  border-radius: 5px;

  cursor: pointer;
}

.btn-guardar {
  border: none;

  background: #222;

  color: white;

  padding: 10px 15px;

  border-radius: 5px;

  cursor: pointer;
}


/* RESPONSIVE */

@media (max-width: 800px) {

  .tarjetas {
    grid-template-columns: 1fr 1fr;
  }

}

@media (max-width: 600px) {

  .header {
    flex-direction: column;

    align-items: flex-start;

    gap: 18px;
  }

  .header .btn-nuevo {
    width: 100%;
  }

  .tarjetas {
    grid-template-columns: 1fr;
  }

  .fila {
    grid-template-columns: 1fr;
  }

}

</style>
```

Con este código ya tienes una base bastante creíble para continuar en clase: **`ref()`, `v-model`, `v-if`, `v-for`, `@click`, `@submit` y `useLocalStorage`**, pero todavía faltan partes importantes del CRUD.

Después puedes ir agregando **Editar → Eliminar → confirmación → validaciones → estadísticas**, en ese orden, para que el avance se vea progresivo.
