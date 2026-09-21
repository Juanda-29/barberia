<script setup>
import { ref, computed, watch } from 'vue'
import { useLocalStorage } from '@vueuse/core'
import Swal from 'sweetalert2'

// ============================================================
// PERSISTENCIA
// ============================================================
const servicios = useLocalStorage('don_ramiro_barberia_db', [])

const catalogoServicios = useLocalStorage('don_ramiro_catalogo_servicios', [
  { id: 1, nombre: 'Corte clásico con tijera', precio: 18000, tipo: 'corte', categoria: 'corte', incluye: [] },
  { id: 2, nombre: 'Low Fade', precio: 25000, tipo: 'corte', categoria: 'corte', incluye: [] },
  { id: 3, nombre: 'Mid Fade', precio: 25000, tipo: 'corte', categoria: 'corte', incluye: [] },
  { id: 4, nombre: 'High Fade', precio: 27000, tipo: 'corte', categoria: 'corte', incluye: [] },
  { id: 5, nombre: 'Taper Fade', precio: 25000, tipo: 'corte', categoria: 'corte', incluye: [] },
  { id: 6, nombre: 'Corte Buzz Cut', precio: 22000, tipo: 'corte', categoria: 'corte', incluye: [] },
  {
    id: 7,
    nombre: 'Corte + Barba',
    precio: 35000,
    tipo: 'corte-barba',
    categoria: 'corte',
    incluye: ['Arreglo y perfilado de barba']
  },
  { id: 8, nombre: 'Arreglo y perfilado de barba', precio: 15000, tipo: 'barba', categoria: 'adicional', incluye: [] },
  { id: 9, nombre: 'Perfilado de cejas', precio: 7000, tipo: 'complementario', categoria: 'adicional', incluye: [] },
  { id: 10, nombre: 'Diseño de línea', precio: 5000, tipo: 'complementario', categoria: 'adicional', incluye: [] }
])

const cierresCaja = useLocalStorage('don_ramiro_cierres_caja', [])

const listaBarberos = [
  { nombre: 'Don Ramiro', comision: 50 },
  { nombre: 'Empleado 1', comision: 40 },
  { nombre: 'Empleado 2', comision: 40 }
]

const barberos = listaBarberos.map(b => b.nombre)
const metodosPago = ['Efectivo', 'Transferencia', 'Tarjeta']
const estadosPago = ['Pagado', 'Abonado', 'Pendiente']

// ============================================================
// ESTADOS DE INTERFAZ
// ============================================================
const mostrarModal = ref(false)
const editandoId = ref(null)
const mostrarModalEliminar = ref(false)
const idAEliminar = ref(null)
const mostrarModalCalificar = ref(false)
const calificandoId = ref(null)
const registrandoCliente = ref(false)
const mostrarSelectorServicios = ref(false)
const mostrarModalCaja = ref(false)
const mostrarModalCatalogo = ref(false)
const servicioCatalogoEditando = ref(null)

const calificacionesTemp = ref({})
const observacionesTemp = ref({})
const erroresFinal = ref({})
const calificacionAbierta = ref({})

// Búsqueda y orden
const busquedaCliente = ref('')
const criterioOrden = ref('fecha-desc')
const ordenPor = ref('fecha')

// Fidelidad
const alertaFrecuenteVisible = ref(false)
const conteoClienteActual = ref(0)
const descuentoFidelidadActivo = ref(false)
const ultimoClienteFrecuenteAvisado = ref('')

// Errores generales
const errores = ref({})

// ============================================================
// FORMULARIO PRINCIPAL
// ============================================================
function obtenerFormVacio() {
  return {
    cliente: '',
    serviciosSeleccionados: [],
    barbero: '',
    fecha: '',
    hora: '',
    precio: 0,
    propina: '',
    metodoPago: '',
    estado: '',
    estadoPago: '',
    abono: '',
    fotoAntes: '',
    fotoDespues: ''
  }
}

const form = ref(obtenerFormVacio())

// ============================================================
// FORMULARIO DEL CATÁLOGO
// ============================================================
function obtenerFormCatalogoVacio() {
  return {
    id: null,
    nombre: '',
    precio: '',
    categoria: 'adicional',
    incluye: []
  }
}

const formCatalogo = ref(obtenerFormCatalogoVacio())

// ============================================================
// NORMALIZACIÓN DEL CATÁLOGO
// Permite trabajar tanto con el formato nuevo (tipo) como con
// el formato anterior (categoria) sin romper el localStorage.
// ============================================================
function categoriaDesdeTipo(tipo) {
  return tipo === 'corte' || tipo === 'corte-barba' ? 'corte' : 'adicional'
}

function tipoDesdeCategoria(categoria) {
  return categoria === 'corte' ? 'corte' : 'complementario'
}

function normalizarCatalogo() {
  catalogoServicios.value = catalogoServicios.value.map((item, index) => {
    const tipo = item.tipo || tipoDesdeCategoria(item.categoria)
    const categoria = item.categoria || categoriaDesdeTipo(tipo)

    return {
      id: item.id ?? Date.now() + index,
      nombre: item.nombre,
      precio: Number(item.precio || 0),
      tipo,
      categoria,
      incluye: Array.isArray(item.incluye)
        ? [...item.incluye]
        : (tipo === 'corte-barba'
            ? ['Arreglo y perfilado de barba']
            : [])
    }
  })
}

normalizarCatalogo()

// ============================================================
// FECHA Y HORA
// ============================================================
function obtenerFechaHoyLocal() {
  const hoy = new Date()
  const anio = hoy.getFullYear()
  const mes = String(hoy.getMonth() + 1).padStart(2, '0')
  const dia = String(hoy.getDate()).padStart(2, '0')
  return `${anio}-${mes}-${dia}`
}

function obtenerHoraActualLocal() {
  const hoy = new Date()
  return `${String(hoy.getHours()).padStart(2, '0')}:${String(hoy.getMinutes()).padStart(2, '0')}`
}

function obtenerHoraMinimaActual() {
  const hora = new Date().getHours()

  // Las citas se manejan por horas predeterminadas (08:00, 09:00,
  // 10:00, etc.). No se exige que coincidan los minutos actuales.
  if (hora < 8) return '08:00'
  if (hora > 20) return '20:00'

  return `${String(hora).padStart(2, '0')}:00`
}

function fechaEsAnteriorAHoy(fecha) {
  return fecha < obtenerFechaHoyLocal()
}

function fechaMinima() {
  return obtenerFechaHoyLocal()
}

function horaMinima() {
  // No se impone una hora mínima.
  // El usuario puede escoger cualquier hora.
  return ''
}

// ============================================================
// SERVICIOS
// ============================================================
function obtenerTipoServicio(nombreServicio) {
  const item = catalogoServicios.value.find(s => s.nombre === nombreServicio)

  if (item?.tipo) return item.tipo
  if (item?.categoria === 'corte') return 'corte'

  const nombre = String(nombreServicio || '').toLowerCase()

  if (
    nombre.includes('corte') ||
    nombre.includes('fade') ||
    nombre.includes('buzz') ||
    nombre.includes('taper')
  ) {
    return 'corte'
  }

  if (nombre.includes('barba')) {
    return nombre.includes('corte') ? 'corte-barba' : 'barba'
  }

  return 'complementario'
}

function esCorte(nombreServicio) {
  const tipo = obtenerTipoServicio(nombreServicio)
  return tipo === 'corte' || tipo === 'corte-barba'
}

function esServicioConBarba(nombreServicio) {
  return obtenerTipoServicio(nombreServicio) === 'corte-barba'
}

function esArregloBarba(nombreServicio) {
  return obtenerTipoServicio(nombreServicio) === 'barba'
}

function obtenerItemCatalogo(nombreServicio) {
  return catalogoServicios.value.find(s => s.nombre === nombreServicio)
}

function puedeSeleccionarServicio(nombreServicio) {
  const seleccionados = form.value.serviciosSeleccionados

  if (seleccionados.includes(nombreServicio)) return true

  if (esCorte(nombreServicio)) {
    for (let i = 0; i < seleccionados.length; i++) {
      if (esCorte(seleccionados[i])) return false
    }
  }

  if (
    esServicioConBarba(nombreServicio) &&
    seleccionados.includes('Arreglo y perfilado de barba')
  ) {
    return false
  }

  if (
    esArregloBarba(nombreServicio) &&
    seleccionados.some(nombre => esServicioConBarba(nombre))
  ) {
    return false
  }

  return true
}

function estaSeleccionado(nombreServicio) {
  return form.value.serviciosSeleccionados.includes(nombreServicio)
}

function corteDeshabilitado(item) {
  if (estaSeleccionado(item.nombre)) return false

  return form.value.serviciosSeleccionados.some(nombre => esCorte(nombre))
}

function itemIncluidoEnCorteActual(nombreServicio) {
  const corteActual = form.value.serviciosSeleccionados.find(nombre => esCorte(nombre))
  if (!corteActual) return false

  const item = obtenerItemCatalogo(corteActual)
  return Array.isArray(item?.incluye) && item.incluye.includes(nombreServicio)
}

function toggleTipo(item) {
  const nombre = item.nombre
  const seleccionados = [...form.value.serviciosSeleccionados]
  const indice = seleccionados.indexOf(nombre)

  if (indice !== -1) {
    seleccionados.splice(indice, 1)
    form.value.serviciosSeleccionados = seleccionados
    actualizarPrecioFormulario()
    return
  }

  if (!puedeSeleccionarServicio(nombre)) {
    errores.value = {
      ...errores.value,
      servicios: esCorte(nombre)
        ? 'Solo puede seleccionar un corte.'
        : 'Ese servicio ya está incluido en el corte seleccionado.'
    }
    return
  }

  form.value.serviciosSeleccionados = [...seleccionados, nombre]
  errores.value = { ...errores.value, servicios: '' }
  actualizarPrecioFormulario()
}

function obtenerServiciosDelRegistro(servicio) {
  if (Array.isArray(servicio.servicios) && servicio.servicios.length > 0) {
    return [...servicio.servicios]
  }

  return servicio.tipoServicio ? [servicio.tipoServicio] : []
}

function calcularTotalBaseServicios(serviciosSeleccionados) {
  return serviciosSeleccionados.reduce((total, nombre) => {
    const item = obtenerItemCatalogo(nombre)
    return total + Number(item?.precio || 0)
  }, 0)
}

function calcularDescuentoFidelidad() {
  if (!descuentoFidelidadActivo.value) return 0
  return Math.round(calcularTotalBaseServicios(form.value.serviciosSeleccionados) * 0.10)
}

function calcularTotalFormulario() {
  const base = calcularTotalBaseServicios(form.value.serviciosSeleccionados)
  return Math.max(0, base - calcularDescuentoFidelidad())
}

function calcularTotalConPropina() {
  return calcularTotalFormulario() + Number(form.value.propina || 0)
}

function calcularSaldoFormulario() {
  const total = calcularTotalConPropina()
  const abono = Number(form.value.abono || 0)
  return Math.max(0, total - abono)
}

function actualizarPrecioFormulario() {
  form.value.precio = calcularTotalFormulario()
}

function listaServicios(servicio) {
  return obtenerServiciosDelRegistro(servicio).join(', ')
}

// ============================================================
// HISTORIAL Y ESTADÍSTICAS
// ============================================================
function contarServiciosCliente(nombre) {
  const nombreNormalizado = String(nombre || '').trim().toLowerCase()
  if (!nombreNormalizado) return 0

  return servicios.value.filter(
    s => String(s.cliente || '').trim().toLowerCase() === nombreNormalizado
  ).length
}

function calcularGastoTotalCliente(nombre) {
  const nombreNormalizado = String(nombre || '').trim().toLowerCase()

  return servicios.value
    .filter(
      s => String(s.cliente || '').trim().toLowerCase() === nombreNormalizado
    )
    .reduce(
      (total, s) =>
        total +
        Number(
          s.totalConPropina ??
          (Number(s.precio || 0) + Number(s.propina || 0))
        ),
      0
    )
}

function totalServicio(servicio) {
  return Number(
    servicio.totalConPropina ??
    (Number(servicio.precio || 0) + Number(servicio.propina || 0))
  )
}

function obtenerServiciosFiltrados() {
  let lista = servicios.value.filter(s => !s.archivado)

  if (busquedaCliente.value.trim()) {
    const busqueda = busquedaCliente.value.trim().toLowerCase()
    lista = lista.filter(
      s => String(s.cliente || '').toLowerCase().includes(busqueda)
    )
  }

  if (ordenPor.value === 'precio') {
    lista.sort((a, b) => totalServicio(b) - totalServicio(a))
  } else if (ordenPor.value === 'calificacion') {
    lista.sort(
      (a, b) => Number(b.calificacion || 0) - Number(a.calificacion || 0)
    )
  } else {
    lista.sort((a, b) =>
      `${b.fecha || ''} ${b.hora || ''}`.localeCompare(
        `${a.fecha || ''} ${a.hora || ''}`
      )
    )
  }

  return lista
}

function calcularIngresosTotales() {
  return servicios.value
    .filter(s => !s.archivado)
    .reduce((total, s) => total + Number(s.abono || 0), 0)
}

function contarPagados() {
  return servicios.value.filter(
    s => !s.archivado && obtenerEstadoServicio(s) === 'Pagado'
  ).length
}

function contarPendientes() {
  return servicios.value.filter(
    s => !s.archivado && obtenerEstadoServicio(s) === 'Pendiente'
  ).length
}

function contarAbonados() {
  return servicios.value.filter(
    s =>
      !s.archivado &&
      obtenerEstadoServicio(s) === 'Abonado' &&
      calcularSaldoRegistro(s) > 0
  ).length
}

function totalServicios() {
  return servicios.value.filter(s => !s.archivado).length
}

function totalVentas() {
  return servicios.value
    .filter(s => !s.archivado && obtenerEstadoServicio(s) === 'Pagado')
    .reduce((total, s) => total + totalServicio(s), 0)
}

function totaldebe() {
  return servicios.value
    .filter(s => !s.archivado)
    .reduce((total, s) => total + calcularSaldoRegistro(s), 0)
}

function calcularPromedioCalificacion() {
  const calificadas = servicios.value.filter(
    s => !s.archivado && Number(s.calificacion || 0) > 0
  )

  if (calificadas.length === 0) return 0

  const total = calificadas.reduce(
    (sum, s) => sum + Number(s.calificacion || 0),
    0
  )

  return total / calificadas.length
}

function obtenerBarberoDestacado() {
  const conteo = {}

  servicios.value
    .filter(s => !s.archivado)
    .forEach(s => {
      if (!conteo[s.barbero]) conteo[s.barbero] = 0
      conteo[s.barbero]++
    })

  let nombre = 'Sin registros'
  let mayor = 0

  Object.keys(conteo).forEach(nombreBarbero => {
    if (conteo[nombreBarbero] > mayor) {
      mayor = conteo[nombreBarbero]
      nombre = nombreBarbero
    }
  })

  return nombre
}

function obtenerNombreTurno(hora) {
  if (!hora) return ''
  const h = Number(String(hora).split(':')[0])

  if (h < 12) return 'Mañana'
  if (h < 18) return 'Tarde'
  return 'Noche'
}

function obtenerClaseTurno(hora) {
  if (!hora) return ''
  const h = Number(String(hora).split(':')[0])

  if (h < 12) return 'shift-morning'
  if (h < 18) return 'shift-afternoon'
  return 'shift-night'
}

const serviciosVisibles = computed(() => obtenerServiciosFiltrados())
const hayServiciosVisibles = computed(() => serviciosVisibles.value.length > 0)

const gruposPorTurno = computed(() => {
  const grupos = {
    Mañana: [],
    Tarde: [],
    Noche: []
  }

  serviciosVisibles.value.forEach(servicio => {
    const turno = obtenerNombreTurno(servicio.hora) || 'Tarde'
    grupos[turno].push(servicio)
  })

  return Object.keys(grupos)
    .filter(nombre => grupos[nombre].length > 0)
    .map(nombre => ({
      nombre,
      servicios: grupos[nombre]
    }))
})

const historialCliente = computed(() => {
  const nombre = busquedaCliente.value.trim()
  if (!nombre) return null

  return {
    veces: contarServiciosCliente(nombre),
    total: calcularGastoTotalCliente(nombre)
  }
})

const estadisticasHoy = computed(() => {
  const hoy = obtenerFechaHoyLocal()
  const lista = servicios.value.filter(
    s => s.fecha === hoy && !s.archivado
  )

  const totalVendidoHoy = lista.reduce(
    (total, s) => total + obtenerMontoCobrado(s),
    0
  )

  const calificadas = lista.filter(s => Number(s.calificacion || 0) > 0)
  const promedioCalificacion = calificadas.length
    ? calificadas.reduce(
        (sum, s) => sum + Number(s.calificacion || 0),
        0
      ) / calificadas.length
    : 0

  const conteo = {}
  lista.forEach(s => {
    conteo[s.barbero] = (conteo[s.barbero] || 0) + 1
  })

  let barberoTop = ''
  let maxCortes = 0

  Object.keys(conteo).forEach(nombre => {
    if (conteo[nombre] > maxCortes) {
      maxCortes = conteo[nombre]
      barberoTop = nombre
    }
  })

  return {
    totalVendidoHoy,
    cantidadHoy: lista.length,
    promedioCalificacion,
    barberoTop,
    maxCortes
  }
})

const comisionesHoy = computed(() => {
  const resultado = {}

  barberos.forEach(nombre => {
    resultado[nombre] = calcularComisionBarbero(nombre)
  })

  return resultado
})

// ============================================================
// FIDELIDAD
// ============================================================
function verificarClienteFrecuente() {
  if (editandoId.value) return

  const nombre = form.value.cliente.trim()

  if (nombre.length < 2) {
    conteoClienteActual.value = 0
    descuentoFidelidadActivo.value = false
    alertaFrecuenteVisible.value = false
    actualizarPrecioFormulario()
    return
  }

  const conteo = contarServiciosCliente(nombre)
  const esFrecuente = conteo >= 5

  conteoClienteActual.value = conteo
  descuentoFidelidadActivo.value = esFrecuente
  alertaFrecuenteVisible.value = esFrecuente

  if (
    esFrecuente &&
    ultimoClienteFrecuenteAvisado.value !== nombre.toLowerCase()
  ) {
    ultimoClienteFrecuenteAvisado.value = nombre.toLowerCase()

    Swal.fire({
      icon: 'success',
      title: '¡Cliente frecuente!',
      text: '¡Cliente frecuente, aplica 10% de descuento!',
      confirmButtonText: 'Aceptar',
      confirmButtonColor: '#2f9e44'
    })
  }

  actualizarPrecioFormulario()
}

// Comprueba automáticamente el cliente mientras se escribe.
// La alerta solo aparece una vez por cliente dentro del formulario actual.
watch(
  () => form.value.cliente,
  () => {
    verificarClienteFrecuente()
  }
)

// ============================================================
// ABRIR / EDITAR / CERRAR
// ============================================================
function abrirNuevoModal() {
  registrandoCliente.value = false
  form.value = obtenerFormVacio()
  editandoId.value = null
  errores.value = {}
  alertaFrecuenteVisible.value = false
  descuentoFidelidadActivo.value = false
  conteoClienteActual.value = 0
  ultimoClienteFrecuenteAvisado.value = ''
  mostrarSelectorServicios.value = false
  mostrarModal.value = true
}

function abrirModalNuevo() {
  abrirNuevoModal()
}

function editarServicio(servicio) {
  if (servicioYaFinalizo(servicio)) return

  registrandoCliente.value = false

  const estado = obtenerEstadoServicio(servicio)

  form.value = {
    cliente: servicio.cliente || '',
    serviciosSeleccionados: obtenerServiciosDelRegistro(servicio),
    barbero: servicio.barbero || '',
    fecha: servicio.fecha || '',
    hora: servicio.hora || '',
    precio: Number(servicio.precio || 0),
    propina: servicio.propina || '',
    metodoPago: servicio.metodoPago || '',
    estado,
    estadoPago: estado,
    abono: servicio.abono || '',
    fotoAntes: servicio.fotoAntes || '',
    fotoDespues: servicio.fotoDespues || ''
  }

  editandoId.value = servicio.id
  errores.value = {}
  mostrarSelectorServicios.value = false
  descuentoFidelidadActivo.value = Boolean(servicio.descuentoFidelidad)
  alertaFrecuenteVisible.value = false
  mostrarModal.value = true
}

function cerrarModal() {
  registrandoCliente.value = false
  mostrarModal.value = false
  errores.value = {}
  mostrarSelectorServicios.value = false
  alertaFrecuenteVisible.value = false
  descuentoFidelidadActivo.value = false
  conteoClienteActual.value = 0
  ultimoClienteFrecuenteAvisado.value = ''
  form.value = obtenerFormVacio()
}

function cancelar() {
  cerrarModal()
}

// ============================================================
// VALIDACIONES DE FECHA Y HORA
// ============================================================
function validarFechaYHora(err) {
  const hoy = obtenerFechaHoyLocal()

  if (!form.value.fecha) {
    err.fecha = 'Seleccione una fecha.'
  } else if (form.value.fecha < hoy) {
    err.fecha = 'La fecha no puede ser anterior a hoy.'
  }

  if (!form.value.hora) {
    err.hora = 'Seleccione una hora.'
    return
  }

  // La hora es completamente libre.
  // Se puede registrar el corte en cualquier hora disponible,
  // incluso si la hora ya pasó. Esto permite registrar servicios
  // reales que se hicieron fuera del horario de agenda o cargar
  // servicios históricos.
  //
  // No se compara contra la hora actual y no se limita a 08:00-20:00.
  // El input permite cualquier hora y cualquier minuto.
}

// ============================================================
// VALIDACIÓN GENERAL DEL FORMULARIO
// ============================================================
function validarFormulario() {
  const err = {}

  if (
    !form.value.cliente ||
    form.value.cliente.trim().length < 2
  ) {
    err.cliente = 'Debe ingresar el nombre del cliente.'
  }

  if (
    !form.value.serviciosSeleccionados ||
    form.value.serviciosSeleccionados.length === 0
  ) {
    err.servicios = 'Debe seleccionar al menos un servicio.'
  }

  if (!form.value.barbero) {
    err.barbero = 'Seleccione el barbero que atendió.'
  }

  validarFechaYHora(err)

  const propina = Number(form.value.propina || 0)

  if (
    form.value.propina !== '' &&
    (Number.isNaN(propina) || propina < 0)
  ) {
    err.propina = 'La propina debe ser un valor válido de 0 o mayor.'
  }

  const estado = form.value.estadoPago || form.value.estado

  if (!form.value.metodoPago) {
    err.metodoPago = 'Seleccione el método de pago.'
  }

  if (!estado) {
    err.estado = 'Seleccione el estado del pago.'
  }

  const total = calcularTotalConPropina()

  if (total <= 0) {
    err.servicios = 'Seleccione servicios para calcular el total.'
  }

  if (estado === 'Abonado') {
    const valorAbono = String(form.value.abono ?? '').trim()
    const abono = Number(valorAbono)

    if (
      valorAbono === '' ||
      !/^\d+$/.test(valorAbono) ||
      !Number.isFinite(abono) ||
      abono <= 0
    ) {
      err.abono = 'Ingrese un valor de abono válido.'
    } else if (abono >= total) {
      err.abono =
        'El abono debe ser menor que el total. Si paga todo, seleccione Pagado.'
    }
  }

  if (estado === 'Pagado') {
    form.value.abono = String(total)
  }

  if (estado === 'Pendiente') {
    form.value.abono = '0'
  }

  form.value.estado = estado
  form.value.estadoPago = estado

  errores.value = err

  return Object.keys(err).filter(k => err[k]).length === 0
}

// ============================================================
// FOTOS BASE64
// ============================================================
function leerFotoBase64(evento, campo) {
  const archivo = evento.target.files && evento.target.files[0]
  if (!archivo) return

  if (!archivo.type.startsWith('image/')) {
    errores.value = {
      ...errores.value,
      fotos: 'Seleccione un archivo de imagen.'
    }
    evento.target.value = ''
    return
  }

  if (archivo.size > 800 * 1024) {
    errores.value = {
      ...errores.value,
      fotos:
        'Cada foto debe pesar máximo 800 KB para cuidar localStorage.'
    }
    evento.target.value = ''
    return
  }

  const lector = new FileReader()

  lector.onload = () => {
    form.value[campo] = lector.result
    errores.value = {
      ...errores.value,
      fotos: ''
    }
  }

  lector.readAsDataURL(archivo)
}

function manejarFoto(evento, campo) {
  leerFotoBase64(evento, campo)
}

function quitarFoto(campo) {
  form.value[campo] = ''
}

function cargarFotoAntes(evento) {
  leerFotoBase64(evento, 'fotoAntes')
}

function cargarFotoDespues(evento) {
  leerFotoBase64(evento, 'fotoDespues')
}

// ============================================================
// GUARDAR SERVICIO
// ============================================================
function guardarServicio() {
  if (registrandoCliente.value) return
  if (!validarFormulario()) return

  registrandoCliente.value = true

  const total = calcularTotalFormulario()
  const propina = Number(form.value.propina || 0)
  const totalConPropina = total + propina
  const estado = form.value.estadoPago || form.value.estado

  const abono =
    estado === 'Abonado'
      ? Number(form.value.abono)
      : estado === 'Pagado'
        ? totalConPropina
        : 0

  const descuento = calcularDescuentoFidelidad()
  const precioBase = calcularTotalBaseServicios(
    form.value.serviciosSeleccionados
  )

  const datos = {
    cliente: form.value.cliente.trim(),
    servicios: [...form.value.serviciosSeleccionados],
    tipoServicio: form.value.serviciosSeleccionados[0] || '',
    barbero: form.value.barbero,
    fecha: form.value.fecha,
    hora: form.value.hora,
    horaFin: obtenerHoraFin(form.value.fecha, form.value.hora),
    precio: total,
    precioBase,
    descuento,
    descuentoFidelidad: descuento > 0,
    propina,
    totalConPropina,
    metodoPago: form.value.metodoPago,
    estado,
    estadoPago: estado,
    abono,
    fotoAntes: form.value.fotoAntes || '',
    fotoDespues: form.value.fotoDespues || '',
    comisionPorcentaje: obtenerComisionPorcentaje(form.value.barbero)
  }

  setTimeout(() => {
    if (editandoId.value) {
      const idx = servicios.value.findIndex(
        s => s.id === editandoId.value
      )

      if (idx !== -1) {
        servicios.value[idx] = {
          ...servicios.value[idx],
          ...datos
        }
      }
    } else {
      servicios.value.push({
        id: Date.now(),
        ...datos,
        calificacion: 0,
        calificado: false,
        observaciones: '',
        observacionesFinales: '',
        estadoServicio: 'En espera',
        archivado: false
      })
    }

    registrandoCliente.value = false
    cerrarModal()
  }, 300)
}

// ============================================================
// ESTADO DEL PAGO Y SALDOS
// ============================================================
function obtenerEstadoServicio(servicio) {
  return servicio.estadoPago || servicio.estado || 'Pendiente'
}

function calcularSaldoRegistro(servicio) {
  const total = totalServicio(servicio)
  return Math.max(0, total - Number(servicio.abono || 0))
}

function obtenerMontoCobrado(servicio) {
  const estado = obtenerEstadoServicio(servicio)

  if (estado === 'Pagado') return totalServicio(servicio)
  if (estado === 'Abonado') return Number(servicio.abono || 0)

  return 0
}

// ============================================================
// BLOQUEO FINAL Y ESTADO DEL SERVICIO
// ============================================================
function obtenerHoraFin(fecha, hora) {
  if (!fecha || !hora) return ''

  const [horas, minutos] = hora.split(':').map(Number)
  const fechaHora = new Date(
    fecha + 'T' +
    String(horas).padStart(2, '0') +
    ':' +
    String(minutos || 0).padStart(2, '0')
  )

  fechaHora.setMinutes(fechaHora.getMinutes() + 60)

  return `${String(fechaHora.getHours()).padStart(2, '0')}:${String(
    fechaHora.getMinutes()
  ).padStart(2, '0')}`
}

function servicioEnCurso(servicio) {
  if (!servicio.fecha || !servicio.hora) return false
  if (servicio.calificado === true) return false

  const ahora = new Date()
  const inicio = new Date(`${servicio.fecha}T${servicio.hora}:00`)
  const horaFin = servicio.horaFin || obtenerHoraFin(servicio.fecha, servicio.hora)
  const fin = new Date(`${servicio.fecha}T${horaFin}:00`)

  return ahora >= inicio && ahora < fin
}

function servicioYaTermino(servicio) {
  if (!servicio.fecha || !servicio.hora) return false

  const ahora = new Date()
  const horaFin =
    servicio.horaFin ||
    obtenerHoraFin(servicio.fecha, servicio.hora)

  const fin = new Date(`${servicio.fecha}T${horaFin}:00`)

  return ahora >= fin
}

function servicioYaFinalizo(servicio) {
  if (!servicio || !servicio.fecha || !servicio.hora) return false

  // Al terminar el bloque de atención se bloquean definitivamente
  // los botones de Editar y Eliminar.
  return servicioYaTermino(servicio)
}

function puedeEditarOEliminar(servicio) {
  return !servicioYaFinalizo(servicio)
}


async function confirmarEliminacion(servicio) {
  if (!puedeEditarOEliminar(servicio)) return

  const resultado = await Swal.fire({
    title: '¿Eliminar servicio?',
    html: `Se eliminará el servicio de <strong>${String(servicio.cliente || '').replace(/[<>]/g, '')}</strong>.`,
    icon: 'warning',
    showCancelButton: true,
    confirmButtonText: 'Sí, eliminar',
    cancelButtonText: 'Cancelar',
    reverseButtons: true,
    confirmButtonColor: '#d33'
  })

  if (!resultado.isConfirmed) return

  servicios.value = servicios.value.filter(s => s.id !== servicio.id)

  await Swal.fire({
    title: 'Servicio eliminado',
    text: 'El registro fue eliminado correctamente.',
    icon: 'success',
    confirmButtonText: 'Aceptar',
    confirmButtonColor: '#2f9e44'
  })
}

// ============================================================
// CALIFICACIÓN / FINALIZACIÓN
// ============================================================
function generarEstrellas(calificacion) {
  const n = Math.max(0, Math.min(5, Number(calificacion || 0)))
  return '★'.repeat(n) + '☆'.repeat(5 - n)
}

function abrirSeccionCalificacion(servicio) {
  calificacionAbierta.value = {
    ...calificacionAbierta.value,
    [servicio.id]: true
  }

  if (!calificacionesTemp.value[servicio.id]) {
    calificacionesTemp.value = {
      ...calificacionesTemp.value,
      [servicio.id]: 5
    }
  }
}

function finalizarServicio(servicio) {
  const calificacion = Number(
    calificacionesTemp.value[servicio.id] || 0
  )

  const observacion = String(
    observacionesTemp.value[servicio.id] || ''
  ).trim()

  if (calificacion < 1 || calificacion > 5) {
    erroresFinal.value = {
      ...erroresFinal.value,
      [servicio.id]: 'Seleccione una calificación de 1 a 5 estrellas.'
    }
    return
  }

  const idx = servicios.value.findIndex(
    s => s.id === servicio.id
  )

  if (idx === -1) return

  servicios.value[idx] = {
    ...servicios.value[idx],
    calificacion,
    calificado: true,
    observaciones: observacion,
    observacionesFinales: observacion,
    estadoServicio: 'Finalizado'
  }

  delete erroresFinal.value[servicio.id]
  delete calificacionesTemp.value[servicio.id]
  delete observacionesTemp.value[servicio.id]

  calificacionAbierta.value = {
    ...calificacionAbierta.value,
    [servicio.id]: false
  }
}

// Compatibilidad con el modal de calificación del script nuevo
const formCalificacion = ref({
  calificacion: 5,
  observaciones: ''
})

function abrirModalCalificar(servicio) {
  calificandoId.value = servicio.id
  formCalificacion.value = {
    calificacion: servicio.calificacion || 5,
    observaciones:
      servicio.observacionesFinales ||
      servicio.observaciones ||
      ''
  }
  mostrarModalCalificar.value = true
}

function cerrarModalCalificar() {
  mostrarModalCalificar.value = false
  calificandoId.value = null
}

function guardarCalificacion() {
  const idx = servicios.value.findIndex(
    s => s.id === calificandoId.value
  )

  if (idx !== -1) {
    servicios.value[idx].calificacion = Number(
      formCalificacion.value.calificacion
    )
    servicios.value[idx].observaciones =
      formCalificacion.value.observaciones.trim()
    servicios.value[idx].observacionesFinales =
      formCalificacion.value.observaciones.trim()
    servicios.value[idx].calificado = true
    servicios.value[idx].estadoServicio = 'Finalizado'
  }

  cerrarModalCalificar()
}

// ============================================================
// CIERRE DE CAJA
// ============================================================
function obtenerServiciosDelDia() {
  return servicios.value.filter(
    s => s.fecha === obtenerFechaHoyLocal() && !s.archivado
  )
}

function calcularCajaDelDia() {
  const resumen = {
    efectivo: 0,
    transferencia: 0,
    tarjeta: 0,
    pendiente: 0
  }

  obtenerServiciosDelDia().forEach(servicio => {
    const cobrado = obtenerMontoCobrado(servicio)

    if (servicio.metodoPago === 'Efectivo') {
      resumen.efectivo += cobrado
    }

    if (servicio.metodoPago === 'Transferencia') {
      resumen.transferencia += cobrado
    }

    if (servicio.metodoPago === 'Tarjeta') {
      resumen.tarjeta += cobrado
    }

    resumen.pendiente += calcularSaldoRegistro(servicio)
  })

  return resumen
}

function obtenerComisionPorcentaje(nombre) {
  const barbero = listaBarberos.find(
    b => b.nombre === nombre
  )

  return barbero ? Number(barbero.comision) : 0
}

function calcularComisionBarbero(nombre) {
  return obtenerServiciosDelDia()
    .filter(s => s.barbero === nombre)
    .reduce((total, s) => {
      const porcentaje = Number(
        s.comisionPorcentaje ??
        obtenerComisionPorcentaje(nombre)
      )

      return (
        total +
        Number(s.precio || 0) * porcentaje / 100
      )
    }, 0)
}

function abrirModalCaja() {
  mostrarModalCaja.value = true
}

function cerrarModalCaja() {
  mostrarModalCaja.value = false
}

function cerrarCajaYArchivar() {
  const delDia = obtenerServiciosDelDia()

  // Se calculan antes de archivar para no perder las comisiones del día.
  const resumen = calcularCajaDelDia()

  const comisiones = listaBarberos.map(b => ({
    barbero: b.nombre,
    porcentaje: b.comision,
    valor: calcularComisionBarbero(b.nombre)
  }))

  delDia.forEach(servicio => {
    const idx = servicios.value.findIndex(
      s => s.id === servicio.id
    )

    if (idx !== -1) {
      servicios.value[idx].archivado = true
    }
  })

  cierresCaja.value.push({
    id: Date.now(),
    fecha: obtenerFechaHoyLocal(),
    ...resumen,
    comisiones
  })

  cerrarModalCaja()
}

async function cerrarCaja() {
  const serviciosHoy = obtenerServiciosDelDia()

  if (!serviciosHoy.length) {
    await Swal.fire({
      title: 'Caja sin servicios',
      text: 'No hay servicios registrados para cerrar la caja de hoy.',
      icon: 'info',
      confirmButtonText: 'Aceptar',
      confirmButtonColor: '#2f9e44'
    })
    return
  }

  const resumen = calcularCajaDelDia()
  const comisiones = listaBarberos.map(b => ({
    barbero: b.nombre,
    porcentaje: b.comision,
    valor: calcularComisionBarbero(b.nombre)
  }))

  const totalCobrado =
    Number(resumen.efectivo || 0) +
    Number(resumen.transferencia || 0) +
    Number(resumen.tarjeta || 0)

  const confirmacion = await Swal.fire({
    title: '¿Cerrar caja del día?',
    html: `
      <div style="text-align:left;line-height:1.7">
        <strong>Servicios:</strong> ${serviciosHoy.length}<br>
        <strong>Efectivo:</strong> $${Number(resumen.efectivo).toLocaleString('es-CO')}<br>
        <strong>Transferencia:</strong> $${Number(resumen.transferencia).toLocaleString('es-CO')}<br>
        <strong>Tarjeta:</strong> $${Number(resumen.tarjeta).toLocaleString('es-CO')}<br>
        <strong>Total cobrado:</strong> $${totalCobrado.toLocaleString('es-CO')}<br>
        <strong>Pendiente:</strong> $${Number(resumen.pendiente).toLocaleString('es-CO')}
      </div>
    `,
    icon: 'question',
    showCancelButton: true,
    confirmButtonText: 'Cerrar caja',
    cancelButtonText: 'Cancelar',
    reverseButtons: true,
    confirmButtonColor: '#2f9e44'
  })

  if (!confirmacion.isConfirmed) return

  cerrarCajaYArchivar()

  const comisionesHtml = comisiones
    .map(c => `<div><strong>${c.barbero} (${c.porcentaje}%):</strong> $${Number(c.valor).toLocaleString('es-CO')}</div>`)
    .join('')

  await Swal.fire({
    title: 'Caja cerrada correctamente',
    html: `
      <div style="text-align:left;line-height:1.7">
        <strong>Resumen del día</strong><br><br>
        Servicios: ${serviciosHoy.length}<br>
        Efectivo: $${Number(resumen.efectivo).toLocaleString('es-CO')}<br>
        Transferencia: $${Number(resumen.transferencia).toLocaleString('es-CO')}<br>
        Tarjeta: $${Number(resumen.tarjeta).toLocaleString('es-CO')}<br>
        <strong>Total cobrado: $${totalCobrado.toLocaleString('es-CO')}</strong><br>
        Pendiente: $${Number(resumen.pendiente).toLocaleString('es-CO')}
        <hr>
        <strong>Comisiones</strong><br>
        ${comisionesHtml || 'Sin comisiones registradas.'}
      </div>
    `,
    icon: 'success',
    confirmButtonText: 'Aceptar',
    confirmButtonColor: '#2f9e44'
  })
}

// ============================================================
// CATÁLOGO EDITABLE
// ============================================================
const adicionalesDisponibles = computed(() =>
  catalogoServicios.value.filter(
    item => item.categoria === 'adicional'
  )
)

function abrirModalCatalogo() {
  servicioCatalogoEditando.value = null
  formCatalogo.value = obtenerFormCatalogoVacio()
  errores.value = {}
  mostrarModalCatalogo.value = true
}

function abrirCatalogo() {
  abrirModalCatalogo()
}

function cerrarModalCatalogo() {
  mostrarModalCatalogo.value = false
  servicioCatalogoEditando.value = null
  formCatalogo.value = obtenerFormCatalogoVacio()
  errores.value = {}
}

function editarServicioCatalogo(item) {
  servicioCatalogoEditando.value = item.id

  formCatalogo.value = {
    id: item.id,
    nombre: item.nombre,
    precio: item.precio,
    categoria: item.categoria || categoriaDesdeTipo(item.tipo),
    incluye: Array.isArray(item.incluye)
      ? [...item.incluye]
      : []
  }

  errores.value = {}
}

function toggleIncluido(nombre) {
  const lista = [...formCatalogo.value.incluye]
  const indice = lista.indexOf(nombre)

  if (indice !== -1) {
    lista.splice(indice, 1)
  } else {
    lista.push(nombre)
  }

  formCatalogo.value.incluye = lista
}

function guardarServicioCatalogo() {
  const nombre = String(
    formCatalogo.value.nombre || ''
  ).trim()

  const precio = Number(
    formCatalogo.value.precio
  )

  if (!nombre) {
    errores.value = {
      catalogo: 'Ingrese el nombre del servicio.'
    }
    return
  }

  if (Number.isNaN(precio) || precio <= 0) {
    errores.value = {
      catalogo:
        'Ingrese un precio base mayor que 0.'
    }
    return
  }

  const existe = catalogoServicios.value.some(
    s =>
      s.nombre.toLowerCase() ===
        nombre.toLowerCase() &&
      s.id !== formCatalogo.value.id
  )

  if (existe) {
    errores.value = {
      catalogo: 'Ese servicio ya existe.'
    }
    return
  }

  const nombreNormalizado = nombre.toLowerCase()

  const tipo =
    formCatalogo.value.categoria === 'corte'
      ? (nombreNormalizado.includes('barba') ? 'corte-barba' : 'corte')
      : (nombreNormalizado.includes('barba') ? 'barba' : 'complementario')

  const datos = {
    id:
      formCatalogo.value.id ??
      Date.now(),
    nombre,
    precio,
    categoria: formCatalogo.value.categoria,
    tipo,
    incluye:
      formCatalogo.value.categoria === 'corte'
        ? [...formCatalogo.value.incluye]
        : []
  }

  if (formCatalogo.value.id) {
    const idx = catalogoServicios.value.findIndex(
      s => s.id === formCatalogo.value.id
    )

    if (idx !== -1) {
      catalogoServicios.value[idx] = {
        ...catalogoServicios.value[idx],
        ...datos
      }
    }
  } else {
    catalogoServicios.value.push(datos)
  }

  errores.value = {}
  servicioCatalogoEditando.value = null
  formCatalogo.value = obtenerFormCatalogoVacio()
}

function eliminarServicioCatalogo(id) {
  if (catalogoServicios.value.length <= 1) return

  catalogoServicios.value =
    catalogoServicios.value.filter(
      item => item.id !== id
    )
}

// ============================================================
// MENSAJE GENERAL DE ERROR PARA EL DISEÑO ACTUAL
// ============================================================
const errorFormulario = computed(() => {
  const valores = Object.values(errores.value)
    .filter(Boolean)

  return valores.length
    ? valores[0]
    : ''
})
</script>

<template>

  <div class="pagina">

    <header class="header">

      <div class="header-interior">

        <div>

          <h1>
            BARBERIA DON RAMIRO
          </h1>

          <p>
            CONTROL DE SERVICIOS
          </p>

        </div>

        <button
          class="boton-nuevo"
          @click="abrirModalNuevo"
        >
          + NUEVO SERVICIO
        </button>

      </div>

    </header>

    <main class="contenedor">

      <section class="resumen">

        <div class="resumen-item">

          <span class="numero">
            {{ totalServicios() }}
          </span>

          <span class="texto">
            SERVICIOS
          </span>

        </div>

        <div class="resumen-item">

          <span class="numero">
            {{ contarPagados() }}
          </span>

          <span class="texto">
            PAGADOS
          </span>

        </div>

        <div class="resumen-item">

          <span class="numero">
            {{ contarPendientes() }}
          </span>

          <span class="texto">
            PENDIENTES
          </span>

        </div>

        <div class="resumen-item">

          <span class="numero">
            {{ contarAbonados() }}
          </span>

          <span class="texto">
            ABONADOS
          </span>

        </div>

        <div class="resumen-item">

          <span class="numero">
            ${{ totalVentas().toLocaleString('es-CO') }}
          </span>

          <span class="texto">
            VENTAS PAGADAS
          </span>

        </div>

        <div class="resumen-item">

          <span class="numero">
            ${{ totaldebe().toLocaleString('es-CO') }}
          </span>

          <span class="texto">
            TOTAL DEBE
          </span>

        </div>

      </section>

      <section class="panel-herramientas">

        <button
          class="boton-secundario"
          @click="abrirCatalogo"
        >
          Gestionar servicios
        </button>

        <button
          class="boton-secundario"
          @click="cerrarCaja"
        >
          Cerrar caja del día
        </button>

      </section>

      <section class="panel-estadisticas">

        <h2 class="titulo-seccion">
          ESTADÍSTICAS DE HOY
        </h2>

        <div class="stats-grid">

          <div class="stat-box">

            <span class="numero">
              ${{ estadisticasHoy.totalVendidoHoy.toLocaleString('es-CO') }}
            </span>

            <span class="texto">
              VENDIDO HOY
            </span>

          </div>

          <div class="stat-box">

            <span class="numero">
              {{ estadisticasHoy.cantidadHoy }}
            </span>

            <span class="texto">
              SERVICIOS HOY
            </span>

          </div>

          <div class="stat-box">

            <span class="numero">
              {{ estadisticasHoy.promedioCalificacion.toFixed(1) }}
            </span>

            <span class="texto">
              PROMEDIO CALIFICACIÓN
            </span>

          </div>

          <div class="stat-box">

            <span class="numero">
              {{ estadisticasHoy.barberoTop || '—' }}
            </span>

            <span class="texto">
              BARBERO DEL DÍA
              ({{ estadisticasHoy.maxCortes }})
            </span>

          </div>

        </div>

      </section>

      <section
        v-if="
          Object.values(comisionesHoy)
            .some(v => v > 0)
        "
        class="panel-comisiones"
      >

        <h2 class="titulo-seccion">
          COMISIONES DE HOY
        </h2>

        <div
          v-for="barbero in barberos"
          :key="barbero"
          class="comision-item"
        >

          <span>
            {{ barbero }}
          </span>

          <strong>
            ${{ comisionesHoy[barbero].toLocaleString('es-CO') }}
          </strong>

        </div>

      </section>

      <section class="panel-historial">

        <h2 class="titulo-seccion">
          HISTORIAL POR CLIENTE
        </h2>

        <input
          v-model="busquedaCliente"
          type="text"
          class="input-historial"
          placeholder="Escribe el nombre del cliente..."
        />

        <p
          v-if="
            busquedaCliente.trim() &&
            historialCliente
          "
          class="resultado-cliente"
        >

          Ha venido
          {{ historialCliente.veces }}

          {{
            historialCliente.veces === 1
              ? 'vez'
              : 'veces'
          }}

          y ha gastado un total de
          ${{ historialCliente.total.toLocaleString('es-CO') }}

        </p>

      </section>

      <div class="titulo-con-orden">

        <h2 class="titulo-seccion">
          SERVICIOS REGISTRADOS
        </h2>

        <div class="controles-orden">

          <button
            :class="[
              'boton-orden',
              {
                activo:
                  ordenPor === 'fecha'
              }
            ]"
            @click="
              ordenPor = 'fecha'
            "
          >
            Fecha
          </button>

          <button
            :class="[
              'boton-orden',
              {
                activo:
                  ordenPor === 'precio'
              }
            ]"
            @click="
              ordenPor = 'precio'
            "
          >
            Precio
          </button>

          <button
            :class="[
              'boton-orden',
              {
                activo:
                  ordenPor === 'calificacion'
              }
            ]"
            @click="
              ordenPor = 'calificacion'
            "
          >
            Calificación
          </button>

        </div>

      </div>

      <section
        v-if="!hayServiciosVisibles"
        class="vacio"
      >

        <p>
          No hay servicios registrados todavía..
        </p>

      </section>

      <section
        v-if="hayServiciosVisibles"
        class="lista-turnos"
      >

        <div
          v-for="grupo in gruposPorTurno"
          :key="grupo.nombre"
          class="grupo-turno"
        >

          <h3 class="turno-header">
            {{ grupo.nombre }}
          </h3>

          <div class="lista">

            <article
              v-for="servicio in grupo.servicios"
              :key="servicio.id"
              class="tarjeta"
            >

              <div class="tarjeta-top">

                <h3>
                  {{ servicio.cliente }}
                </h3>

                <strong class="precio">
                  ${{ totalServicio(servicio).toLocaleString('es-CO') }}
                </strong>

              </div>

              <p
                v-if="
                  servicio.propina > 0
                "
                class="linea propina-linea"
              >
                ${{ Number(servicio.precio).toLocaleString('es-CO') }}
                +
                ${{ Number(servicio.propina).toLocaleString('es-CO') }}
                propina
              </p>

              <p class="linea">
                {{ listaServicios(servicio) }}
              </p>

              <p class="linea">
                Atendido por
                {{ servicio.barbero }}
              </p>

              <p class="linea">
                {{ servicio.fecha }}
                ·
                {{ servicio.hora }}
              </p>

              <p class="linea">
                Termina:
                {{
                  servicio.horaFin ||
                  obtenerHoraFin(
                    servicio.fecha,
                    servicio.hora
                  )
                }}
              </p>

              <p class="linea">
                Pago:
                {{ servicio.metodoPago }}
              </p>

              <p class="linea">

                <span
                  v-if="
                    servicio.estadoPago ===
                    'Pagado'
                  "
                  class="estado ok"
                >
                  Pagado
                </span>

                <span
                  v-else-if="
                    servicio.estadoPago ===
                    'Pendiente'
                  "
                  class="estado pendiente"
                >
                  Pendiente
                </span>

                <span
                  v-else
                  class="estado abonado"
                >
                  Abonado
                </span>

              </p>

              <!-- RESUMEN DEL DINERO -->
              <div class="resumen-pago">
                <div class="resumen-pago-item">
                  <span>Cliente abona</span>
                  <strong>
                    ${{
                      Number(
                        servicio.estadoPago === 'Pagado'
                          ? totalServicio(servicio)
                          : servicio.estadoPago === 'Abonado'
                            ? servicio.abono || 0
                            : 0
                      ).toLocaleString('es-CO')
                    }}
                  </strong>
                </div>

                <div class="resumen-pago-item restante">
                  <span>Restante</span>
                  <strong>
                    ${{
                      calcularSaldoRegistro(servicio).toLocaleString('es-CO')
                    }}
                  </strong>
                </div>
              </div>

              <p class="linea">

                <span
                  v-if="
                    servicio.estadoServicio ===
                    'Finalizado'
                  "
                  class="estado ok"
                >
                  Servicio finalizado
                </span>

                <span
                  v-else-if="
                    servicioEnCurso(
                      servicio
                    )
                  "
                  class="estado pendiente"
                >
                  En curso
                </span>

              </p>

              <p class="linea">
                Calificación:
                <span
                  v-if="servicio.calificado === true"
                  class="estrellas"
                >
                  {{ generarEstrellas(servicio.calificacion) }}
                </span>
                <span v-else class="texto-pendiente">
                  Pendiente al terminar el servicio
                </span>
              </p>

              <p class="linea">
                Observaciones:
                <span v-if="servicio.observacionesFinales || servicio.observaciones">
                  {{ servicio.observacionesFinales || servicio.observaciones }}
                </span>
                <span v-else class="texto-pendiente">
                  Pendientes de registrar
                </span>
              </p>

              <!--
                CALIFICACIÓN:
                Se habilita cuando termina el bloque de atención
                o cuando el servicio ya está pagado.
                El cliente puede registrar calificación y observaciones
                opcionales. Al terminar el servicio se bloquean
                definitivamente Editar y Eliminar.
              -->
              <button
                v-if="
                  (
                    servicioYaTermino(servicio) ||
                    obtenerEstadoServicio(servicio) === 'Pagado'
                  ) &&
                  servicio.calificado !== true
                "
                type="button"
                class="boton-calificar"
                @click="
                  abrirSeccionCalificacion(servicio)
                "
              >
                Calificar servicio
              </button>

              <!--
                FORMULARIO DE CALIFICACIÓN.
                Se muestra al pulsar "Calificar servicio".
              -->
              <div
                v-if="
                  (
                    servicioYaTermino(servicio) ||
                    obtenerEstadoServicio(servicio) === 'Pagado'
                  ) &&
                  servicio.calificado !== true &&
                  calificacionAbierta[servicio.id]
                "
                class="seccion-cierre-servicio"
              >

                <h4 class="subtitulo-cierre">
                  Calificar y observación final (opcional)
                </h4>

                <div class="selector-estrellas">

                  <button
                    v-for="n in 5"
                    :key="n"
                    type="button"
                    :class="[
                      'estrella-boton',
                      {
                        activa:
                          (
                            calificacionesTemp[
                              servicio.id
                            ] || 0
                          ) >= n
                      }
                    ]"
                    @click="
                      calificacionesTemp[
                        servicio.id
                      ] = n
                    "
                  >
                    ★
                  </button>

                </div>

                <textarea
                  v-model="
                    observacionesTemp[
                      servicio.id
                    ]
                  "
                  class="observacion-final-input"
                  placeholder="Observaciones finales (opcional)"
                ></textarea>

                <p
                  v-if="
                    erroresFinal[
                      servicio.id
                    ]
                  "
                  class="mensaje-error-final"
                >
                  {{
                    erroresFinal[
                      servicio.id
                    ]
                  }}
                </p>

                <button
                  type="button"
                  class="boton-calificar"
                  @click="
                    finalizarServicio(
                      servicio
                    )
                  "
                >
                  Guardar y finalizar servicio
                </button>

              </div>

              <!--
                Al terminar el servicio:
                Editar y Eliminar quedan bloqueados.
              -->

              <div
                v-if="
                  servicio.calificado === true
                "
                class="calificacion-servicio"
              >

                <span class="estrellas">

                  {{
                    generarEstrellas(
                      servicio.calificacion
                    )
                  }}

                </span>

                <span
                  class="calificacion-bloqueada"
                >
                  Ya calificado
                </span>

              </div>

              <p
                v-if="
                  servicio.calificado === true &&
                  servicio.observacionesFinales
                "
                class="observacion"
              >
                {{ servicio.observacionesFinales }}
              </p>

              <div
                v-if="
                  servicio.fotoAntes ||
                  servicio.fotoDespues
                "
                class="fotos-servicio"
              >

                <div
                  v-if="servicio.fotoAntes"
                  class="foto-item"
                >

                  <span class="foto-label">
                    Antes
                  </span>

                  <img
                    :src="servicio.fotoAntes"
                    class="foto-preview"
                  />

                </div>

                <div
                  v-if="servicio.fotoDespues"
                  class="foto-item"
                >

                  <span class="foto-label">
                    Después
                  </span>

                  <img
                    :src="servicio.fotoDespues"
                    class="foto-preview"
                  />

                </div>

              </div>


              <div class="acciones-servicio">

                <button
                  type="button"
                  class="boton-editar"
                  :disabled="!puedeEditarOEliminar(servicio)"
                  :class="{ bloqueado: !puedeEditarOEliminar(servicio) }"
                  @click="editarServicio(servicio)"
                  :title="
                    puedeEditarOEliminar(servicio)
                      ? 'Editar servicio'
                      : 'Servicio finalizado: edición bloqueada'
                  "
                >
                  Editar
                </button>

                <button
                  type="button"
                  class="boton-eliminar"
                  :disabled="!puedeEditarOEliminar(servicio)"
                  :class="{ bloqueado: !puedeEditarOEliminar(servicio) }"
                  @click="confirmarEliminacion(servicio)"
                  :title="
                    puedeEditarOEliminar(servicio)
                      ? 'Eliminar servicio'
                      : 'Servicio finalizado: eliminación bloqueada'
                  "
                >
                  Eliminar
                </button>

              </div>
            </article>

          </div>

        </div>

      </section>

    </main>

    <div
      v-if="mostrarModal"
      class="modal-fondo"
    >

      <div class="modal">

        <div class="modal-header">

          <h2>
            Nuevo servicio
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
          @submit.prevent="
            guardarServicio
          "
        >

          <div class="campo">

            <label>
              NOMBRE DEL CLIENTE
            </label>

            <input
              v-model="form.cliente"
              type="text"
              @blur="
                verificarClienteFrecuente
              "
            />

          </div>

          <div class="campo">

            <label>
              SERVICIOS
            </label>

            <p class="ayuda-servicios">
              Elige un solo corte.
              Puedes combinarlo con
              los adicionales que quieras.
            </p>

            <div class="grupo-servicios">

              <h4
                class="subtitulo-servicios"
              >
                CORTES (elige solo uno)
              </h4>

              <label
                v-for="
                  tipo in
                  catalogoServicios.filter(
                    t =>
                      t.categoria ===
                      'corte'
                  )
                "
                :key="tipo.id"
                :class="[
                  'opcion-check',
                  {
                    deshabilitada:
                      corteDeshabilitado(
                        tipo
                      )
                  }
                ]"
              >

                <input
                  type="checkbox"
                  :checked="
                    estaSeleccionado(
                      tipo.nombre
                    )
                  "
                  :disabled="
                    corteDeshabilitado(
                      tipo
                    )
                  "
                  @change="
                    toggleTipo(tipo)
                  "
                />

                {{ tipo.nombre }}

                ${{ tipo.precio.toLocaleString('es-CO') }}

              </label>

            </div>

            <div class="grupo-servicios">

              <h4
                class="subtitulo-servicios"
              >
                ADICIONALES
                (puedes combinar varios)
              </h4>

              <label
                v-for="
                  tipo in
                  catalogoServicios.filter(
                    t =>
                      t.categoria ===
                      'adicional'
                  )
                "
                :key="tipo.id"
                :class="[
                  'opcion-check',
                  {
                    deshabilitada:
                      itemIncluidoEnCorteActual(
                        tipo.nombre
                      )
                  }
                ]"
              >

                <input
                  type="checkbox"
                  :checked="
                    estaSeleccionado(
                      tipo.nombre
                    )
                  "
                  :disabled="
                    itemIncluidoEnCorteActual(
                      tipo.nombre
                    )
                  "
                  @change="
                    toggleTipo(tipo)
                  "
                />

                {{ tipo.nombre }}

                ${{ tipo.precio.toLocaleString('es-CO') }}

                <span
                  v-if="
                    itemIncluidoEnCorteActual(
                      tipo.nombre
                    )
                  "
                  class="etiqueta-incluido"
                >
                  incluido en el corte
                </span>

              </label>

            </div>

          </div>

          <div class="campo">

            <label>
              BARBERO
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
                v-for="
                  barbero in barberos
                "
                :key="barbero"
                :value="barbero"
              >
                {{ barbero }}
              </option>

            </select>

          </div>

          <div class="fila">

            <div class="campo">

              <label>
                FECHA
              </label>

              <input
                v-model="form.fecha"
                type="date"
                :min="
                  fechaMinima()
                "
              />

            </div>

            <div class="campo">

              <label>
                HORA
              </label>

              <input
                v-model="form.hora"
                type="time"
                step="60"
                min="00:00"
                max="23:59"
                
              />

            </div>

          </div>

          <div class="campo">

            <label>
              PRECIO TOTAL
            </label>

            <div
              class="
                input-con-simbolo
                precio-bloqueado
              "
            >

              <span class="simbolo">
                $
              </span>

              <span
                class="precio-formateado"
              >
                {{
                  Number(
                    form.precio || 0
                  ).toLocaleString(
                    'es-CO'
                  )
                }}
              </span>

            </div>

          </div>

          <div class="campo">

            <label>
              PROPINA (opcional)
            </label>

            <div
              class="input-con-simbolo"
            >

              <span class="simbolo">
                $
              </span>

              <input
                v-model="
                  form.propina
                "
                type="number"
                min="0"
                placeholder="0"
              />

            </div>

          </div>

          <div class="fila">

            <div class="campo">

              <label>
                METODO DE PAGO
              </label>

              <select
                v-model="
                  form.metodoPago
                "
              >

                <option
                  value=""
                  disabled
                >
                  Seleccione
                </option>

                <option
                  v-for="
                    metodo in
                    metodosPago
                  "
                  :key="metodo"
                  :value="metodo"
                >
                  {{ metodo }}
                </option>

              </select>

            </div>

            <div class="campo">

              <label>
                ESTADO DEL PAGO
              </label>

              <select
                v-model="
                  form.estadoPago
                "
              >

                <option
                  value=""
                  disabled
                >
                  Seleccione
                </option>

                <option
                  v-for="
                    estado in
                    estadosPago
                  "
                  :key="estado"
                  :value="estado"
                >
                  {{ estado }}
                </option>

              </select>

            </div>

          </div>

          <div
            v-if="form.estadoPago === 'Abonado'"
            class="fila fila-abono"
          >

            <div class="campo">

              <label>
                VALOR ABONADO POR EL CLIENTE
              </label>

              <div class="input-con-simbolo">

                <span class="simbolo">
                  $
                </span>

                <input
                  v-model="form.abono"
                  type="text"
                  inputmode="numeric"
                  autocomplete="off"
                  placeholder="Ingrese el valor abonado"
                  @input="form.abono = String(form.abono || '').replace(/\D/g, '')"
                />

              </div>

              <small
                v-if="errores.abono"
                class="texto-error-campo"
              >
                {{ errores.abono }}
              </small>

              <small class="saldo-abono">
                Saldo pendiente: ${{
                  calcularSaldoFormulario().toLocaleString('es-CO')
                }}
              </small>

            </div>

          </div>

          <div class="fila">

            <div class="campo">

              <label>
                FOTO ANTES (opcional)
              </label>

              <input
                type="file"
                accept="image/*"
                @change="
                  manejarFoto(
                    $event,
                    'fotoAntes'
                  )
                "
              />

              <div
                v-if="
                  form.fotoAntes
                "
                class="foto-item"
              >

                <img
                  :src="form.fotoAntes"
                  class="foto-preview"
                />

                <button
                  type="button"
                  class="boton-quitar-foto"
                  @click="
                    quitarFoto(
                      'fotoAntes'
                    )
                  "
                >
                  Quitar
                </button>

              </div>

            </div>

            <div class="campo">

              <label>
                FOTO DESPUÉS (opcional)
              </label>

              <input
                type="file"
                accept="image/*"
                @change="
                  manejarFoto(
                    $event,
                    'fotoDespues'
                  )
                "
              />

              <div
                v-if="
                  form.fotoDespues
                "
                class="foto-item"
              >

                <img
                  :src="
                    form.fotoDespues
                  "
                  class="foto-preview"
                />

                <button
                  type="button"
                  class="boton-quitar-foto"
                  @click="
                    quitarFoto(
                      'fotoDespues'
                    )
                  "
                >
                  Quitar
                </button>

              </div>

            </div>

          </div>

          <div
            v-show="
              errorFormulario
            "
            class="mensaje-error"
          >
            {{ errorFormulario }}
          </div>

          <div
            class="botones-formulario"
          >

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
              Guardar servicio
            </button>

          </div>

        </form>
              </div>

    </div>

    <!-- =========================
         MODAL DEL CATÁLOGO
    ========================= -->

    <div
      v-if="mostrarModalCatalogo"
      class="modal-fondo"
    >

      <div class="modal">

        <div class="modal-header">

          <h2>
            Gestionar servicios
          </h2>

          <button
            class="cerrar"
            @click="
              cerrarModalCatalogo
            "
          >
            ×
          </button>

        </div>

        <div class="formulario">

          <div class="campo">

            <label>
              NOMBRE DEL SERVICIO
            </label>

            <input
              v-model="
                formCatalogo.nombre
              "
              type="text"
            />

          </div>

          <div class="fila">

            <div class="campo">

              <label>
                PRECIO SUGERIDO
              </label>

              <input
                v-model="
                  formCatalogo.precio
                "
                type="number"
                min="0"
              />

            </div>

            <div class="campo">

              <label>
                CATEGORÍA
              </label>

              <select
                v-model="
                  formCatalogo.categoria
                "
              >

                <option value="corte">
                  Corte (solo uno a la vez)
                </option>

                <option value="adicional">
                  Adicional (combinable)
                </option>

              </select>

            </div>

          </div>

          <div
            v-if="
              formCatalogo.categoria ===
                'corte' &&
              adicionalesDisponibles.length > 0
            "
            class="campo"
          >

            <label>
              ESTE CORTE YA INCLUYE
              (no se podrán elegir por aparte)
            </label>

            <label
              v-for="
                adicional in
                adicionalesDisponibles
              "
              :key="adicional.id"
              class="opcion-check"
            >

              <input
                type="checkbox"
                :checked="
                  formCatalogo.incluye.includes(
                    adicional.nombre
                  )
                "
                @change="
                  toggleIncluido(
                    adicional.nombre
                  )
                "
              />

              {{ adicional.nombre }}

            </label>

          </div>

          <button
            type="button"
            class="boton-guardar"
            @click="
              guardarServicioCatalogo
            "
          >
            {{
              formCatalogo.id
                ? 'Guardar cambios'
                : 'Agregar servicio'
            }}
          </button>

          <div class="lista-catalogo">

            <div
              v-for="
                item in catalogoServicios
              "
              :key="item.id"
              class="catalogo-item"
            >

              <span>

                {{ item.nombre }}

                ·

                ${{ item.precio.toLocaleString('es-CO') }}

                ·

                {{
                  item.categoria ===
                  'corte'
                    ? 'Corte'
                    : 'Adicional'
                }}

              </span>

              <!--
                ESTOS BOTONES SON DEL CATÁLOGO,
                NO DE LOS SERVICIOS REGISTRADOS.
                SE CONSERVAN.
              -->

              <div
                class="catalogo-acciones"
              >

                <button
                  type="button"
                  class="boton-editar"
                  @click="
                    editarServicioCatalogo(
                      item
                    )
                  "
                >
                  Editar
                </button>

                <button
                  type="button"
                  class="boton-eliminar"
                  @click="
                    eliminarServicioCatalogo(
                      item.id
                    )
                  "
                >
                  Eliminar
                </button>

              </div>

            </div>

          </div>

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
  font-family:
    Arial,
    Helvetica,
    sans-serif;
}

/* =========================
   HEADER
========================= */

.header {
  background: #7e5a3d;
  color: #fff;
  border-radius: 15px;
  background-image:
    url('/barberia.jpg');
  background-size: cover;
  background-position: center;
  min-height: 300px;
}

.header-interior {
  width: 100%;
  min-height: 450px;
  padding:
    20px
    50px
    220px
    70px;

  display: flex;
  align-items: center;
  justify-content: space-between;

  gap: 30px;

  background:
    rgba(0, 0, 0, 0.40);

  border-radius: 15px;
}

.header h1 {
  margin: 0;
  font-size: 48px;
  font-weight: bold;
  letter-spacing: 2px;
  line-height: 1.1;
}

.header p {
  margin:
    12px
    0
    0;

  color: #b3d35d;
  font-size: 24px;
  font-weight: bold;
  letter-spacing: 2px;
}

.boton-nuevo {
  border: none;
  background: #31af5b;
  color: #fff;

  padding:
    15px
    22px;

  border-radius: 20px;
  font-size: 14px;
  cursor: pointer;
  white-space: nowrap;
}

.boton-nuevo:hover {
  background: #111;
}

/* =========================
   CONTENEDOR
========================= */

.contenedor {
  width: 92%;
  max-width: 1000px;
  margin:
    24px
    auto
    50px;
}

/* =========================
   RESUMEN
========================= */

.resumen {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
  margin-bottom: 24px;
}

.resumen-item {
  background: #c9e7e9;
  border: 1px solid #ddd;
  border-radius: 20px;

  padding:
    12px
    16px;

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

/* =========================
   HERRAMIENTAS
========================= */

.panel-herramientas {
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
  margin-bottom: 24px;
}

.boton-secundario {
  border: none;
  background: #1f1f1f;
  color: #fff;

  padding:
    10px
    16px;

  border-radius: 15px;
  font-size: 13px;
  cursor: pointer;
}

.boton-secundario:hover {
  background: #31af5b;
}

.panel-estadisticas,
.panel-comisiones,
.panel-historial {
  background: #fff;
  border: 1px solid #ddd;
  border-radius: 15px;

  padding: 16px;

  margin-bottom: 20px;
}

.stats-grid {
  display: grid;
  grid-template-columns:
    repeat(4, 1fr);

  gap: 12px;
}

.stat-box {
  background: #c9e7e9;
  border-radius: 15px;
  padding: 12px;
  text-align: center;
}

.stat-box .numero {
  display: block;
  font-size: 16px;
  font-weight: bold;
}

.stat-box .texto {
  font-size: 11px;
  color: #666;
}

.comision-item {
  display: flex;
  justify-content: space-between;

  padding:
    8px
    0;

  border-bottom:
    1px solid #eee;

  font-size: 14px;
}

.input-historial {
  width: 100%;
  padding: 8px;

  border:
    1px
    solid
    #ccc;

  border-radius: 8px;
  font-size: 14px;
}

.resultado-cliente {
  margin-top: 10px;
  font-size: 14px;
  color: #2f6b3a;
  font-weight: bold;
}

/* =========================
   ORDEN Y TURNOS
========================= */

.titulo-con-orden {
  display: flex;
  justify-content: space-between;
  align-items: center;

  flex-wrap: wrap;
  gap: 10px;
}

.controles-orden {
  display: flex;
  gap: 8px;
}

.boton-orden {
  border:
    1px
    solid
    #ccc;

  background: #fff;

  padding:
    6px
    12px;

  border-radius: 12px;
  font-size: 12px;
  cursor: pointer;
}

.boton-orden.activo {
  background: #31af5b;
  color: #fff;
  border-color: #31af5b;
}

.lista-turnos {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.turno-header {
  font-size: 16px;
  color: #7e5a3d;

  border-bottom:
    2px
    solid
    #e0dec3;

  padding-bottom: 6px;
  margin-bottom: 12px;
}

/* =========================
   VACÍO
========================= */

.vacio {
  background: #fff;
  border: 1px solid #ddd;
  border-radius: 4px;

  padding: 30px;

  text-align: center;
}

/* =========================
   TARJETAS
========================= */

.lista {
  display: grid;

  grid-template-columns:
    repeat(3, 1fr);

  gap: 14px;
}

.tarjeta {
  background: #e0dec3;
  border:
    1px
    solid
    #b39393;

  border-radius: 30px;

  padding: 18px;
}

.tarjeta-top {
  display: flex;
  justify-content: space-between;
  align-items: baseline;

  margin-bottom: 8px;
}

.tarjeta-top h3 {
  margin: 0;
  font-size: 19px;
  font-weight: bold;
}

.precio {
  font-size: 17px;
  font-weight: bold;
}

.propina-linea {
  font-size: 13px;
  color: #b8860b;
  font-weight: bold;
}

.linea {
  margin:
    6px
    0;

  font-size: 15px;
  color: #444;

  display: flex;
  justify-content: space-between;
  align-items: center;
}

/* =========================
   ESTADOS
========================= */

.estado {
  font-size: 13px;
  font-weight: bold;

  padding:
    4px
    8px;

  border-radius: 5px;
}

.estado.ok {
  background: #e4f2e6;
  color: #2f6b3a;
}

.estado.pendiente {
  background: #fdf1dc;
  color: #8a5a13;}

.estado.abonado {
  background: #f7ddcc;
  color: #d17812;
}

/* =========================
   RESUMEN DE PAGO
========================= */

.resumen-pago {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 8px;
  margin: 10px 0;
  padding: 10px;
  border-radius: 10px;
  background: #f5f4e8;
  border: 1px solid #c8c5a7;
}

.resumen-pago-item {
  display: flex;
  flex-direction: column;
  gap: 3px;
}

.resumen-pago-item span {
  font-size: 12px;
  color: #555;
  font-weight: 600;
}

.resumen-pago-item strong {
  font-size: 16px;
  color: #222;
}

.resumen-pago-item.restante strong {
  color: #b3261e;
}

/* =========================
   ESTRELLAS
========================= */

.estrellas {
  color: #b8860b;
  font-size: 18px;
}

/* =========================
   CALIFICACIÓN
========================= */

.calificacion-servicio {
  display: flex;
  align-items: center;
  justify-content: space-between;

  gap: 8px;

  margin-top: 12px;
  padding-top: 10px;

  border-top:
    1px
    solid
    rgba(0, 0, 0, 0.12);
}

.seccion-cierre-servicio {
  margin-top: 12px;
  padding-top: 10px;

  border-top:
    1px
    solid
    rgba(0, 0, 0, 0.12);

  display: flex;
  flex-direction: column;
  gap: 8px;
}

.subtitulo-cierre {
  margin: 0;

  font-size: 13px;

  color: #7e5a3d;

  font-weight: bold;
}

.selector-estrellas {
  display: flex;
  gap: 4px;
}

.estrella-boton {
  border: none;
  background: none;

  font-size: 22px;

  color: #ccc;

  cursor: pointer;

  padding: 0;

  line-height: 1;
}

.estrella-boton.activa {
  color: #b8860b;
}

.observacion-final-input {
  width: 100%;

  border:
    1px
    solid
    #ccc;

  border-radius: 8px;

  padding:
    6px
    8px;

  font-size: 13px;

  resize: vertical;

  min-height: 50px;

  background: #fff;
}

.mensaje-error-final {
  background: #fbe6e6;
  color: #8c3535;

  padding:
    6px
    8px;

  border-radius: 4px;

  font-size: 11px;

  margin: 0;
}

.boton-calificar {
  border: none;

  background: #d6a72c;

  color: #fff;

  padding:
    8px
    12px;

  border-radius: 12px;

  font-size: 12px;
  font-weight: bold;

  cursor: pointer;

  transition:
    background 0.15s ease,
    transform 0.15s ease;
}

.boton-calificar:hover {
  background: #b8860b;
  transform: translateY(-1px);
}

.calificacion-bloqueada {
  font-size: 12px;

  color: #666;

  background: #e5e5e5;

  padding:
    7px
    10px;

  border-radius: 12px;

  font-weight: bold;
}

/* =========================
   OBSERVACIÓN
========================= */

.observacion {
  margin-top: 10px;

  font-size: 14px;

  color: #555;

  background: #e0dec3;

  padding:
    7px
    9px;

  border-radius: 4px;
}

/* =========================
   FOTOS
========================= */

.fotos-servicio {
  display: flex;
  gap: 10px;
  margin-top: 10px;
}

.foto-item {
  display: flex;
  flex-direction: column;
  align-items: center;

  gap: 4px;
}

.foto-label {
  font-size: 11px;
  color: #666;
  font-weight: bold;
}

.foto-preview {
  width: 80px;
  height: 80px;

  object-fit: cover;

  border-radius: 8px;

  border:
    1px
    solid
    #ccc;
}

.boton-quitar-foto {
  border: none;

  background: #e0b7b7;

  color: #a03a3a;

  border-radius: 8px;

  padding:
    4px
    8px;

  font-size: 11px;

  cursor: pointer;

  margin-top: 4px;
}

/* =========================
   MODAL
========================= */

.modal-fondo {
  position: fixed;

  inset: 0;

  background:
    rgba(0, 0, 0, 0.5);

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

  border-radius: 15px;
}

.modal-header {
  display: flex;

  justify-content: space-between;

  align-items: center;

  padding:
    14px
    16px;

  border-bottom:
    1px
    solid
    #eee;
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

  grid-template-columns:
    1fr
    1fr;

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

  border-bottom:
    1px
    solid
    #ccc;

  padding:
    6px
    2px;

  font-size: 13px;

  background: transparent;
}

.campo input:focus,
.campo select:focus,
.campo textarea:focus {
  outline: none;

  border-bottom-color:
    #333;
}

.campo textarea {
  height: 55px;

  resize: vertical;
}

.precio-formateado {
  font-size: 13px;

  color: #555;

  width: 100%;
}

.ayuda-servicios {
  font-size: 11px;

  color: #888;

  margin:
    0
    0
    4px;

  font-weight: normal;
}

/* =========================
   PRECIO
========================= */

.input-con-simbolo {
  display: flex;

  align-items: center;

  border-bottom:
    1px
    solid
    #ccc;

  padding:
    6px
    2px;
}

.input-con-simbolo:focus-within {
  border-bottom-color:
    #333;
}

.input-con-simbolo .simbolo {
  font-size: 13px;

  color: #444;

  margin-right: 4px;
}

.input-con-simbolo input {
  border: none !important;

  padding: 0 !important;

  width: 100%;

  background: transparent;
}

.input-con-simbolo input:focus {
  outline: none;
}

.precio-bloqueado {
  background: #f2f2f2;

  border-radius: 5px;
}

.precio-bloqueado input {
  cursor: not-allowed;

  color: #555;
}

/* =========================
   CHECKBOX
========================= */

.grupo-servicios {
  background: #f7f7f7;

  border-radius: 10px;

  padding:
    8px
    10px;

  margin-top: 6px;
}

.subtitulo-servicios {
  margin:
    0
    0
    6px;

  font-size: 11px;

  color: #7e5a3d;

  letter-spacing: 1px;
}

.opcion-check {
  display: flex;

  align-items: center;

  gap: 6px;

  font-size: 13px;

  font-weight: normal;

  color: #333;

  padding:
    3px
    0;
}

.opcion-check input {
  width: auto;
}

.opcion-check.deshabilitada {
  color: #999;

  opacity: 0.6;
}

.opcion-check input:disabled {
  cursor: not-allowed;
}

.etiqueta-incluido {
  font-size: 10px;

  color: #b8860b;

  font-style: italic;

  margin-left: 4px;
}

/* =========================
   ERROR
========================= */

.mensaje-error {
  background: #fbe6e6;

  color: #8c3535;

  padding: 8px;

  border-radius: 4px;

  font-size: 12px;
}

/* =========================
   BOTONES FORMULARIO
========================= */

.botones-formulario {
  display: grid;

  grid-template-columns:
    1fr
    1fr;

  gap: 10px;

  margin-top: 4px;
}

.boton-cancelar,
.boton-guardar {
  padding: 9px;

  border-radius: 15px;

  cursor: pointer;

  font-weight: bold;
}

.boton-cancelar {
  border:
    1px
    solid
    #c9c9c9;

  background: #eeeeee;
  color: #444;
}

.boton-cancelar:hover {
  background: #dddddd;
}

.boton-guardar {
  border: none;

  background: #31af5b;

  color: #fff;
}

.boton-guardar:hover {
  background: #26964b;
}

/* =========================
   CATÁLOGO
========================= */

.lista-catalogo {
  display: flex;

  flex-direction: column;

  gap: 8px;

  margin-top: 10px;
}

.catalogo-item {
  display: flex;

  justify-content: space-between;

  align-items: center;

  gap: 8px;

  background: #f7f7f7;

  padding:
    8px
    10px;

  border-radius: 8px;

  font-size: 13px;
}

.catalogo-acciones {
  display: flex;

  gap: 6px;
}

.catalogo-acciones button {
  padding:
    5px
    8px;

  font-size: 11px;

  border: none;
  border-radius: 8px;

  cursor: pointer;
}

.catalogo-acciones button:first-child {
  background: #9aa9e8;
  color: #20274d;
}

.catalogo-acciones button:last-child {
  background: #e6a9b1;
  color: #652a31;
}

.catalogo-acciones button:hover {
  filter: brightness(0.94);
}



/* =========================
   ACCIONES DE SERVICIO
========================= */
.acciones-servicio {
  display: flex;
  gap: 8px;
  margin-top: 12px;
}

.acciones-servicio .boton-editar,
.acciones-servicio .boton-eliminar {
  flex: 1;
  min-width: 0;

  border: none;
  padding: 8px 10px;
  border-radius: 12px;

  font-size: 12px;
  font-weight: bold;

  transition:
    transform 0.15s ease,
    opacity 0.15s ease,
    filter 0.15s ease;
}

.acciones-servicio .boton-editar {
  background: #9aa9e8;
  color: #20274d;
}

.acciones-servicio .boton-editar:hover:not(:disabled) {
  background: #7f91dc;
  transform: translateY(-1px);
}

.acciones-servicio .boton-eliminar {
  background: #e6a9b1;
  color: #652a31;
}

.acciones-servicio .boton-eliminar:hover:not(:disabled) {
  background: #d98f99;
  transform: translateY(-1px);
}

.acciones-servicio button.bloqueado,
.acciones-servicio button:disabled {
  opacity: 0.45;
  cursor: not-allowed;
  filter: grayscale(0.4);
  transform: none;
}

.acciones-servicio button:not(:disabled) {
  cursor: pointer;
}

.texto-pendiente {
  color: #888;
  font-style: italic;
}

    .fila-abono {
      margin-top: 4px;
    }

    .saldo-abono {
      display: block;
      margin-top: 6px;
      color: #555;
    }

    .texto-error-campo {
      display: block;
      margin-top: 5px;
      color: #c62828;
      font-size: 12px;
    }

</style>
