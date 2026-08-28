<script setup>
import { ref } from 'vue'
import { useLocalStorage } from '@vueuse/core'

const mostrarModal = ref(false)

const servicios = useLocalStorage('servicios-barberia', [])

const form = ref({
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

const tiposServicio = [
  'Corte clásico',
  'Corte moderno',
  'Barba',
  'Corte + barba',
  'Cejas',
  'Tinte'
]

const barberos = [
  'Don Ramiro',
  'Carlos',
  'Andrés'
]

function abrirModal() {
  mostrarModal.value = true
}

function cerrarModal() {
  mostrarModal.value = false
}

function guardarServicio() {

  if (
    !form.value.cliente ||
    !form.value.servicio ||
    !form.value.barbero
  ) {
    alert('Complete los campos principales')
    return
  }

  const nuevoServicio = {
    id: Date.now(),
    cliente: form.value.cliente,
    servicio: form.value.servicio,
    barbero: form.value.barbero,
    fecha: form.value.fecha,
    hora: form.value.hora,
    precio: form.value.precio,
    metodo: form.value.metodo,
    estado: form.value.estado,
    calificacion: form.value.calificacion,
    observaciones: form.value.observaciones
  }

  servicios.value.push(nuevoServicio)

  form.value = {
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

  cerrarModal()
}
</script>


<template>

  <div class="pagina">


    <header class="header">

      <div class="imagen-header">

        <img
          src="https://www.beautymarket.es/imagen/min35798.jpg"
          alt="Barbería"
        >

        <div class="fondo"></div>

        <div class="contenido-header">

         <div> 
 
            <h1> 
              <span>🪒</span> Barbería Don Ramiro 
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

        </div>

      </div>

    </header>


    <main class="contenedor">

      <section class="introduccion">

        <h2>
          Servicios registrados
        </h2>

        <p>
          En esta sección se pueden consultar los servicios realizados.
        </p>

      </section>


      <section
        v-if="servicios.length === 0"
        class="sin-datos"
      >

        <div class="icono">
          
        </div>

        <h3>
          No hay servicios registrados
        </h3>

        <p>
          Agregue el primer servicio para comenzar.
        </p>

        <button
          class="btn-nuevo"
          @click="abrirModal"
        >
          + Registrar servicio
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

            <div>

              <h3>
                {{ servicio.cliente }}
              </h3>

              <p>
                {{ servicio.barbero }}
              </p>

            </div>

            <strong>
              ${{ servicio.precio }}
            </strong>

          </div>


          <div class="dato">

            <span>
              Servicio
            </span>

            <strong>
              {{ servicio.servicio }}
            </strong>

          </div>


          <div class="datos">

            <div>

              <span>
                Fecha
              </span>

              <strong>
                {{ servicio.fecha }}
              </strong>

            </div>

            <div>

              <span>
                Hora
              </span>

              <strong>
                {{ servicio.hora }}
              </strong>

            </div>

          </div>


          <div
            v-if="servicio.metodo"
            class="dato"
          >

            <span>
              Método de pago
            </span>

            <strong>
              {{ servicio.metodo }}
            </strong>

          </div>


          <div
            v-if="servicio.estado"
            class="estado"
          >

            {{ servicio.estado }}

          </div>


          <div
            v-if="servicio.calificacion"
            class="calificacion"
          >

            ★★★★★

          </div>


          <div
            v-if="servicio.observaciones"
            class="observacion"
          >

            {{ servicio.observaciones }}

          </div>


          <div class="acciones">

            <button class="btn-editar">
              Editar
            </button>

            <button class="btn-eliminar">
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
              Nuevo servicio
            </h2>

            <p>
              Complete los datos del servicio.
            </p>

          </div>

          <button
            class="cerrar"
            @click="cerrarModal"
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
              placeholder="Ej. Juan castro"
            >

          </div>


          <div class="fila">

            <div class="campo">

              <label>
                Tipo de servicio
              </label>

              <select
                v-model="form.servicio"
              >

                <option value="">
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

                <option value="">
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
              >

            </div>


            <div class="campo">

              <label>
                Hora
              </label>

              <input
                v-model="form.hora"
                type="time"
              >

            </div>

          </div>


          <div class="campo">

            <label>
              Precio cobrado
            </label>

            <input
              v-model="form.precio"
              type="number"
              placeholder="20.000"
            >

          </div>


          <div class="fila">

            <div class="campo">

              <label>
                Método de pago
              </label>

              <select
                v-model="form.metodo"
              >

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

              <label>
                Estado del pago
              </label>

              <select
                v-model="form.estado"
              >

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

            <label>
              Calificación
            </label>

            <select
              v-model="form.calificacion"
            >

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

            <label>
              Observaciones
            </label>

            <textarea
              v-model="form.observaciones"
              placeholder="Escribe una observacion..."
            ></textarea>

          </div>


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

.pagina {
  min-height: 100vh;

  background: #f3f1ed;

  color: #292929;

  font-family: Arial, Helvetica, sans-serif;
}


/* HEADER */

.imagen-header {
 

  height: 320px;

  position: relative;

  overflow: hidden;
}

.imagen-header img {
  width: 100%;
  height: 100%;

  object-fit: cover;
}

.fondo {
  position: absolute;

  inset: 0;

  background: rgba(0, 0, 0, .55);
}

.contenido-header {
  position: absolute;

  inset: 0;

  max-width: 1100px;

  margin: auto;

  padding: 35px;

  display: flex;

  align-items: center;

  justify-content: space-between;

  color: white;
}

.contenido-header h1 {
  margin: 0;

  font-size: 29px;
}

.contenido-header p {
  margin-top: 8px;

  color: #ddd;
}


/* BOTON */

.btn-nuevo {
  padding: 11px 17px;

  border: none;

  border-radius: 5px;

  background: #c39a55;

  color: #1e1e1e;

  font-weight: bold;

  cursor: pointer;
}

.btn-nuevo:hover {
  background: #d2aa68;
}


/* CONTENEDOR */

.contenedor {
  width: 92%;

  max-width: 1100px;

  margin: 35px auto;
}


/* INTRODUCCION */

.introduccion {
  margin-bottom: 22px;
}

.introduccion h2 {
  margin-bottom: 5px;
}

.introduccion p {
  margin: 0;

  color: #777;

  font-size: 14px;
}


/* SIN DATOS */

.sin-datos {
  background: white;

  border: 1px solid #ddd8cf;

  border-radius: 8px;

  padding: 55px 20px;

  text-align: center;
}

.icono {
  width: 55px;
  height: 55px;

  margin: auto;

  display: flex;

  align-items: center;

  justify-content: center;

  border-radius: 50%;

  background: #eee7dc;

  color: #a47b38;

  font-size: 24px;
}

.sin-datos h3 {
  margin-bottom: 5px;
}

.sin-datos p {
  color: #777;

  margin-bottom: 20px;
}


/* LISTA */

.lista {
  display: grid;

  grid-template-columns: repeat(3, 1fr);

  gap: 18px;
}


/* TARJETA */

.tarjeta {
  background: white;

  border: 1px solid #ddd8cf;

  border-radius: 8px;

  padding: 18px;
}

.tarjeta-header {
  display: flex;

  justify-content: space-between;

  align-items: center;

  padding-bottom: 14px;

  border-bottom: 1px solid #eee;
}

.tarjeta-header h3 {
  margin: 0;

  font-size: 16px;
}

.tarjeta-header p {
  margin: 4px 0;

  color: #888;

  font-size: 11px;
}

.tarjeta-header strong {
  color: #a47835;
}


/* DATOS */

.dato {
  padding: 13px 0;
}

.dato span,
.datos span {
  display: block;

  color: #999;

  font-size: 9px;

  margin-bottom: 4px;
}

.dato strong,
.datos strong {
  font-size: 12px;
}

.datos {
  display: grid;

  grid-template-columns: 1fr 1fr;

  gap: 10px;

  padding-bottom: 12px;
}


/* ESTADO */

.estado {
  display: inline-block;

  padding: 5px 8px;

  background: #eee;

  border-radius: 4px;

  font-size: 10px;

  margin-bottom: 10px;
}


/* CALIFICACION */

.calificacion {
  color: #c39a55;

  font-size: 15px;

  padding: 5px 0 12px;
}


/* OBSERVACION */

.observacion {
  background: #f5f3ef;

  padding: 9px;

  border-left: 3px solid #c39a55;

  color: #666;

  font-size: 11px;

  margin-bottom: 12px;
}


/* ACCIONES */

.acciones {
  display: grid;

  grid-template-columns: 1fr 1fr;

  gap: 8px;

  margin-top: 8px;
}

.btn-editar,
.btn-eliminar {
  padding: 9px;

  border-radius: 5px;

  font-size: 11px;

  font-weight: bold;

  cursor: pointer;
}

.btn-editar {
  border: 1px solid #ccc;

  background: white;

  color: #555;
}

.btn-eliminar {
  border: 1px solid #e2c5c5;

  background: #fff7f7;

  color: #995050;
}


/* MODAL */

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

  max-height: 90vh;

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
}

.modal-header p {
  margin: 0;

  color: #888;

  font-size: 12px;
}

.cerrar {
  width: 30px;
  height: 30px;

  border: none;

  border-radius: 50%;

  background: #eee;

  font-size: 20px;

  cursor: pointer;
}


/* FORMULARIO */

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

.campo input,
.campo select,
.campo textarea {
  width: 100%;

  padding: 10px;

  border: 1px solid #d5d2cd;

  border-radius: 5px;

  outline: none;

  font-size: 13px;
}

.campo input:focus,
.campo select:focus,
.campo textarea:focus {
  border-color: #bd914d;
}

.campo textarea {
  height: 75px;

  resize: vertical;
}


/* BOTONES FORMULARIO */

.botones {
  display: grid;

  grid-template-columns: 1fr 1fr;

  gap: 10px;

  margin-top: 5px;
}

.btn-cancelar,
.btn-guardar {
  padding: 11px;

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

  background: #292929;

  color: white;
}


/* RESPONSIVE */

@media (max-width: 850px) {

  .lista {
    grid-template-columns: 1fr 1fr;
  }

}


@media (max-width: 600px) {

  .contenido-header {
    flex-direction: column;

    align-items: flex-start;

    justify-content: center;

    gap: 20px;
  }

  .contenido-header h1 {
    font-size: 23px;
  }

  .btn-nuevo {
    width: 100%;
  }

  .lista {
    grid-template-columns: 1fr;
  }

  .fila {
    grid-template-columns: 1fr;
  }

  .botones {
    grid-template-columns: 1fr;
  }

}

</style>