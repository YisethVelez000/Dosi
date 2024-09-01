<template>
  <div id="app">
    <!-- Navbar reutilizado -->
    <NavbarCliente @toggleCart="toggleCart" @irAllogin="irAllogin" />

    <!-- Componente de carrito -->
    <Carrito ref="carrito" />

    <!-- Fondo de página con imagen -->
    <div
      class="bg-cover bg-center"
      style="
        background-image: url('https://img.freepik.com/foto-gratis/fondo_53876-32170.jpg?w=900&t=st=1723158687~exp=1723159287~hmac=ca7360f6388e32be58092afafa63e288d84316777a9651ab19c310a192076f95');
        min-height: 100vh;
      "
    >
      <!-- Contenido centrado verticalmente -->
      <div class="flex items-center justify-center h-full">
        <!-- Contenedor interno para limitar el ancho -->
        <div class="bg-white shadow-md rounded-lg p-4 md:p-6" style="max-width: 1100px; width: 100%">
          <!-- Estructura de grid para la sección principal -->
          <div class="grid grid-cols-3 gap-4">
            <!-- Contenedor lateral izquierdo para imágenes del producto -->
            <div class="col-span-1">
              <h3 class="text-xl font-semibold text-primary mb-4">
                {{ producto?.NombreProductoCatalogo || 'Imágenes del producto' }}
              </h3>
              <div class="flex flex-wrap gap-4 carousel-wrapper">
                <VaCarousel :items="uploadedImages" class="carousel-container" stateful indicators infinite swipable />
              </div>
            </div>

            <!-- Contenedor central para detalles del producto -->
            <div v-if="producto" class="col-span-1">
              <div class="mb-4">
                <h3 class="text-lg font-semibold mb-2">Detalles del producto</h3>
                <div class="border border-gray-200 p-4 rounded-lg">
                  <!-- Tallas disponibles -->
                  <div class="flex flex-col mb-2">
                    <label class="font-semibold">Talla:</label>
                    <div class="flex gap-2 flex-wrap">
                      <div
                        v-for="talla in selectedTallas"
                        :key="talla.IdTalla"
                        :class="['talla-option', { active: productoCliente.Talla === talla.Talla }]"
                        @click="productoCliente.Talla = talla.Talla"
                      >
                        {{ talla.Talla }}
                      </div>
                    </div>
                    <span v-if="validationMessage.talla" class="text-red-500">{{ validationMessage.talla }}</span>
                  </div>

                  <!-- Colores disponibles -->
                  <div class="flex flex-col mb-2">
                    <label class="font-semibold">Color:</label>
                    <div class="flex gap-2 flex-wrap">
                      <div
                        v-for="color in selectedColors"
                        :key="color.IdColor"
                        :class="['color-option', { active: productoCliente.Color === color.Color }]"
                        @click="productoCliente.Color = color.Color"
                      >
                        {{ color.Color }}
                      </div>
                    </div>
                    <span v-if="validationMessage.color" class="text-red-500">{{ validationMessage.color }}</span>
                  </div>

                  <!-- Stock del producto (visualización) -->
                  <div class="flex flex-col">
                    <label class="font-semibold">Cantidad disponible:</label>
                    <span>{{ producto.Stock }}</span>
                  </div>

                  <!-- Precio del producto (nueva sección) -->
                  <div class="flex flex-col mt-4">
                    <label class="font-semibold">Precio del Producto:</label>
                    <span>{{ producto.PrecioProducto }}</span>
                  </div>

                  <!-- Precio total incluyendo estampados, solo si el producto es personalizable -->
                  <div v-if="producto.Personalizable === 'Si'" class="flex flex-col mt-2">
                    <label class="font-semibold">Precio total con estampados:</label>
                    <span>{{ PrecioProductoConPrecioEstampado() }}</span>
                  </div>
                </div>
              </div>
            </div>

            <!-- Contenedor lateral derecho para personalización del estampado -->
            <div class="col-span-1 scrollable-container">
              <div v-if="showAddEstampadoButton" class="mb-4 flex justify-end">
                <button class="btn-black" @click="agregarEstampado">Agregar Estampado</button>
              </div>

              <div v-for="(estampado, index) in productoCliente.Estampados" :key="index" class="mb-4">
                <h3 class="text-lg font-semibold mb-2">Personalización del Estampado {{ index + 1 }}</h3>
                <div class="border border-gray-200 p-4 rounded-lg">
                  <div class="grid grid-cols-1 gap-4">
                    <div v-if="estampado.mostrarCampos">
                      <div class="flex flex-col mb-2">
                        <label class="font-semibold">Tipo de Estampado:</label>
                        <div class="flex gap-2 flex-wrap">
                          <div
                            v-for="tipo in selectedEstampados"
                            :key="tipo.IdEstampado"
                            :class="['estampado-option', { active: estampado.IdEstampado === tipo.IdEstampado }]"
                            @click="
                              () => {
                                estampado.IdEstampado = tipo.IdEstampado
                                estampado.TipoEstampado = tipo.TipoEstampado
                                actualizarUbicaciones(tipo.IdEstampado, index)
                              }
                            "
                          >
                            {{ tipo.TipoEstampado }}
                          </div>
                        </div>
                        <span v-if="validationMessage.estampadoTipo" class="text-red-500">{{
                          validationMessage.estampadoTipo
                        }}</span>
                      </div>

                      <div class="flex flex-col mb-2">
                        <label class="font-semibold">Ubicación del Estampado:</label>
                        <div class="flex gap-2 flex-wrap">
                          <div
                            v-for="ubicacion in estampado.Ubicaciones"
                            :key="ubicacion.IdUbicacion"
                            :class="[
                              'ubicacion-option',
                              { active: estampado.UbicacionSeleccionada === ubicacion.IdUbicacion },
                            ]"
                            @click="estampado.UbicacionSeleccionada = ubicacion.IdUbicacion"
                          >
                            {{ ubicacion.Ubicacion }}
                          </div>
                        </div>
                        <span v-if="validationMessage.estampadoUbicacion" class="text-red-500">{{
                          validationMessage.estampadoUbicacion
                        }}</span>
                      </div>

                      <!-- Imagen del Estampado -->
                      <div class="flex flex-col mb-2">
                        <label class="font-semibold">Imagen del Estampado:</label>
                        <div class="custom-file-input-wrapper">
                          <input type="file" class="custom-file-input" @change="handleFileChange($event, index)" />
                          <button class="btn-black file-btn" @click.prevent="triggerFileInput(index)">
                            Seleccionar archivo
                          </button>
                          <span class="file-selected">{{
                            estampado.ImagenEstampado?.name || 'Ningún archivo seleccionado'
                          }}</span>
                        </div>
                        <span v-if="validationMessage.imagenEstampado" class="text-red-500">{{
                          validationMessage.imagenEstampado
                        }}</span>
                      </div>
                    </div>

                    <div class="flex justify-end">
                      <button
                        class="bg-red-500 hover:bg-red-700 text-white font-bold py-2 px-4 rounded"
                        @click="eliminarEstampado(index)"
                      >
                        Eliminar Estampado
                      </button>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <div class="flex justify-end mt-4">
            <button class="btn-black" @click="agregarAlCarrito">Agregar al carrito</button>
            <VaButton preset="secondary" color="secondary" @click="onClose">Cancelar</VaButton>
          </div>
        </div>
      </div>
    </div>

    <!-- Pie de página -->
    <footer class="footer">
      <RouterLink to="https://www.instagram.com/dlalys_dosii/">Contacto</RouterLink>
      <RouterLink to="/privacy">Política de privacidad</RouterLink>
    </footer>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { useRouter, useRoute } from 'vue-router'
import { ObtenerColores, ObtenerTallas, ObtenerProductoPorId } from '../../data/pages/productos'
import Carrito from '../../components/Carrito.vue'
import NavbarCliente from '../../components/NavbarCliente.vue'
// Importa el componente de carrito

const router = useRouter()
const route = useRoute()
const validationMessage = ref('')
const cartProducts = ref([])

// Referencia al componente del carrito
const carrito = ref(null)

// Función para abrir el cuadro de diálogo de archivo
function triggerFileInput(index: number) {
  const input = document.querySelectorAll('.custom-file-input')[index] as HTMLInputElement
  if (input) input.click()
}

// Funciones para manejar el almacenamiento en localStorage
const saveCartToLocalStorage = (cartProducts) => {
  localStorage.setItem('cart', JSON.stringify(cartProducts))
}

const loadCartFromLocalStorage = () => {
  const savedCart = localStorage.getItem('cart')
  return savedCart ? JSON.parse(savedCart) : []
}

// Cargar el carrito desde localStorage cuando se monte el componente
onMounted(() => {
  cartProducts.value = loadCartFromLocalStorage()
})

// Función para alternar la visibilidad del sidebar del carrito
// const toggleCartSidebar = () => {
//   isCartOpen.value = !isCartOpen.value
// }

const onClose = () => {
  router.push('/home')
}

const toggleCart = () => {
  if (carrito.value) {
    carrito.value.toggleCartSidebar(true) // Utiliza la referencia definida para abrir el sidebar
  }
}
import { jwtDecode } from 'jwt-decode'

const token = localStorage.getItem('token')
let IdUsuario = ''
if (token) {
  const decoded = jwtDecode(token) as any
  IdUsuario = decoded.idUsuario || ''
  localStorage.setItem('IdUsuario', IdUsuario)
  console.log(IdUsuario)
}
const agregarAlCarrito = async () => {
  // Reset de mensajes de validación

  validationMessage.value = {
    talla: '',
    color: '',
    estampadoTipo: '',
    estampadoUbicacion: '',
    imagenEstampado: '',
  }

  // Validaciones de los campos requeridos
  if (!productoCliente.value.Talla) {
    validationMessage.value.talla = 'Por favor, selecciona una talla.'
    return
  }

  if (!productoCliente.value.Color) {
    validationMessage.value.color = 'Por favor, selecciona un color.'
    return
  }

  if (producto.value.Personalizable === 'Si' && productoCliente.value.Estampados.length === 0) {
    validationMessage.value.estampadoTipo = 'Por favor, selecciona al menos un estampado.'
    return
  }

  for (const estampado of productoCliente.value.Estampados) {
    if (!estampado.TipoEstampado) {
      validationMessage.value.estampadoTipo = 'Por favor, selecciona un tipo de estampado.'
      return
    }

    if (!estampado.UbicacionSeleccionada) {
      validationMessage.value.estampadoUbicacion = 'Por favor, selecciona una ubicación para el estampado.'
      return
    }

    if (estampado.IdEstampado === -1) {
      validationMessage.value.estampadoTipo = 'Por favor, selecciona un estampado válido.'
      return
    }

    if (!estampado.Ubicaciones.some((u) => u.IdUbicacion === estampado.UbicacionSeleccionada)) {
      validationMessage.value.estampadoUbicacion = 'Por favor, selecciona una ubicación válida.'
      return
    }
  }

  // Crear FormData para enviar los datos
  const formData = new FormData()
  formData.append('IdUsuario', IdUsuario)
  formData.append('IdProducto', productoCliente.value.IdProducto.toString())
  formData.append('cantidad', productoCliente.value.Cantidad.toString())
  formData.append('Talla', productoCliente.value.Talla)
  formData.append('Color', productoCliente.value.Color)

  // Serializar el array de estampados a un string JSON
  const estampadosData = productoCliente.value.Estampados.map((estampado) => ({
    IdEstampado: estampado.IdEstampado,
    IdUbicacion: estampado.UbicacionSeleccionada,
    TipoEstampado: estampado.TipoEstampado,
  }))

  formData.append('estampados', JSON.stringify(estampadosData))

  // Agregar imágenes de estampados si existen
  productoCliente.value.Estampados.forEach((estampado, index) => {
    if (estampado.ImagenEstampado) {
      formData.append(`imagenEstampado_${index}`, estampado.ImagenEstampado)
    }
  })

  try {
    const response = await fetch('https://apinodedvp.onrender.com/carrito', {
      method: 'POST',
      body: formData,
    })

    if (!response.ok) {
      const errorData = await response.json()
      validationMessage.value = { ...validationMessage.value, general: errorData.message }
      return
    }

    const result = await response.json()
    console.log(result)
    // Si es exitoso, actualizar el carrito
    const nuevoProducto = {
      IdProducto: productoCliente.value.IdProducto,
      NombreProducto: producto.value.NombreProductoCatalogo,
      ImagenProducto: uploadedImages.value[0],
      Color: productoCliente.value.Color,
      Talla: productoCliente.value.Talla,
      Cantidad: productoCliente.value.Cantidad,
      PrecioUnitario: producto.value.PrecioProducto,
      PrecioTotal: PrecioProductoConPrecioEstampado(),
      Estampados: productoCliente.value.Estampados,
    }

    cartProducts.value.push(nuevoProducto)
    saveCartToLocalStorage(cartProducts.value)

    if (carrito.value) {
      carrito.value.updateCartProducts(cartProducts.value)
      carrito.value.toggleCartSidebar(true)
    }
  } catch (error) {
    console.error('Error al agregar al carrito:', error)
    validationMessage.value = {
      ...validationMessage.value,
      general: 'Ocurrió un error al agregar el producto al carrito.',
    }
  }
}

// Estado de los productos y la personalización
const productoCliente = ref({
  IdProducto: -1,
  NombreProducto: '',
  ImagenProducto: '',
  Color: '',
  Talla: '',
  Cantidad: 1,
  PrecioUnitario: 0,
  PrecioTotal: 0,
  Estampados: [],
})

const selectedColors = ref([])
const selectedTallas = ref([])
const selectedEstampados = ref([])
const tallasEnum = ref([])
const uploadedImages = ref([])
const showAddEstampadoButton = ref(false)
const colorsFromAPI = ref([])
const producto = ref(null)
const ubicacionesEstampado = ref([])
const precioEstampado = ref({})

const obtenerColores = async () => {
  try {
    const response = await ObtenerColores()
    colorsFromAPI.value = response.data
  } catch (error) {
    console.error('Error al obtener los colores:', error)
  }
}

const irAllogin = () => {
  router.push('/auth/login')
}

const obtenerEstampados = async (IdEstampado) => {
  try {
    const response = await fetch(`https://apinodedvp.onrender.com/estampados/${IdEstampado}`)

    const estampado = await response.json()

    ubicacionesEstampado.value = estampado.EstampadosUbicaciones.map((ubicacion) => ({
      IdUbicacion: ubicacion.Ubicacione.IdUbicacion,
      Ubicacion: ubicacion.Ubicacione.Ubicacion,
    }))

    precioEstampado.value = estampado.PrecioEstampado
  } catch (error) {
    console.error('Error al obtener el estampado:', error)
  }
}

const PrecioProductoConPrecioEstampado = () => {
  const precioBase = producto.value ? producto.value.PrecioProducto : 0

  const precioEstampados = productoCliente.value.Estampados.reduce((total, estampado) => {
    return total + (estampado.PrecioEstampado || 0)
  }, 0)

  return precioBase + precioEstampados
}

const obtenerTallas = async () => {
  try {
    const response = await ObtenerTallas()
    tallasEnum.value = response.data.map((talla, index) => ({
      IdTalla: index + 1,
      Talla: talla,
    }))
  } catch (error) {
    console.error('Error al obtener las tallas:', error)
  }
}

const actualizarUbicaciones = async (IdEstampado, index) => {
  try {
    const response = await fetch(`https://apinodedvp.onrender.com/estampados/${IdEstampado}`)
    const data = await response.json()

    productoCliente.value.Estampados[index].Ubicaciones = data.EstampadosUbicaciones.map((ubicacion) => ({
      IdUbicacion: ubicacion.Ubicacione.IdUbicacion,
      Ubicacion: ubicacion.Ubicacione.Ubicacion,
    }))

    productoCliente.value.Estampados[index].PrecioEstampado = data.PrecioEstampado || 0
  } catch (error) {
    console.error('Error al actualizar ubicaciones:', error)
  }
}

onMounted(async () => {
  try {
    let idProducto = route.params.IdProducto
    if (Array.isArray(idProducto)) {
      idProducto = idProducto[0]
    }

    const productoData = await ObtenerProductoPorId(idProducto)

    producto.value = {
      IdProducto: productoData.IdProducto,
      NombreProductoCatalogo: productoData.NombreProductoCatalogo,
      FechaRegistro: productoData.FechaRegistro,
      PrecioProducto: productoData.PrecioProducto,
      Personalizable: productoData.Personalizable,
      EstadoProducto: productoData.EstadoProducto,
      IdFichaTecnica: productoData.IdFichaTecnica,
      Producto_has_Colores: productoData.Producto_has_Colores.map((item) => ({
        IdProductoColor: item.IdProductoColor,
        Color: item.Color.Color,
        IdColor: item.Color.IdColor,
      })),
      Producto_has_Tallas: productoData.Producto_has_Tallas.map((item) => ({
        IdProductosTallas: item.IdProductosTallas,
        Talla: item.Talla.Talla,
        IdTalla: item.Talla.IdTalla,
      })),
      Producto_Has_Imagens: productoData.Producto_Has_Imagens.map((item) => ({
        IdProductoImagen: item.IdProductoImagen,
        Imagen: item.Imagen.Imagen,
      })),
      EstampadoProductos: productoData.EstampadoProductos.map((item) => ({
        IdEstampadoProducto: item.IdEstampadoProducto,
        TipoEstampado: item.Estampado.TipoEstampado,
        IdEstampado: item.Estampado.IdEstampado,
      })),
    }

    selectedColors.value = producto.value.Producto_has_Colores.map((item) => ({
      IdColor: item.IdColor,
      Color: item.Color,
    }))
    selectedTallas.value = producto.value.Producto_has_Tallas.map((item) => ({
      IdTalla: item.IdTalla,
      Talla: item.Talla,
    }))

    selectedEstampados.value = producto.value.EstampadoProductos.map((item) => ({
      IdEstampado: item.IdEstampado,
      TipoEstampado: item.TipoEstampado,
    }))

    uploadedImages.value = producto.value.Producto_Has_Imagens.map((item) => item.Imagen)

    showAddEstampadoButton.value = productoData.Personalizable === 'Si'

    for (const estampado of producto.value.EstampadoProductos) {
      await obtenerEstampados(estampado.IdEstampado)
    }

    productoCliente.value = {
      IdProducto: productoData.IdProducto,
      NombreProducto: productoData.NombreProductoCatalogo,
      ImagenProducto: uploadedImages.value[0],
      Color: '',
      Talla: '',
      Cantidad: 1,
      PrecioUnitario: productoData.PrecioProducto,
      PrecioTotal: productoData.PrecioProducto,
      Estampados: [],
    }

    obtenerColores()
    obtenerTallas()
  } catch (error) {
    console.error('Error al cargar los datos del producto:', error)
  }
})

function agregarEstampado() {
  productoCliente.value.Estampados.push({
    IdDetalleEstampado: productoCliente.value.Estampados.length + 1,
    IdPedidoVenta: -1,
    IdEstampado: -1,
    Ubicacion: '',
    TipoEstampado: '',
    ImagenEstampado: '',
    mostrarCampos: true,
  })
}

function handleFileChange(event, index) {
  const fileInput = event.target
  if (fileInput.files && fileInput.files[0]) {
    productoCliente.value.Estampados[index].ImagenEstampado = fileInput.files[0]
  }
}

function eliminarEstampado(index) {
  productoCliente.value.Estampados.splice(index, 1)
}
</script>

<style scoped>
/* Margen superior para separar del Navbar */
.bg-cover {
  background-size: cover;
  background-repeat: no-repeat;
  margin-top: 20px; /* Margen superior añadido */
}

/* Otros estilos ya existentes... */

/* Ajuste del tamaño del contenedor principal */
.max-w-xl {
  max-width: 800px;
  margin: 0 auto;
}

/* Estilo adicional para pantallas grandes */
@media (min-width: 768px) {
  .max-w-xl {
    padding: 20px;
  }
}

/* Contenedor de precios */
.price-container {
  display: flex;
  justify-content: center;
  align-items: center;
}

/* Contenedor de detalles del producto */
.product-details-container {
  display: flex;
  flex-direction: column;
  align-items: center;
}

/* Estilos para opciones de tallas, colores, tipos de estampado, y ubicaciones */
.talla-option,
.color-option,
.estampado-option,
.ubicacion-option {
  border: 1px solid #ddd;
  padding: 10px 20px;
  border-radius: 50px;
  cursor: pointer;
  text-align: center;
  transition:
    background-color 0.3s,
    color 0.3s;
}

.talla-option.active,
.color-option.active,
.estampado-option.active,
.ubicacion-option.active {
  background-color: #333;
  color: #fff;
  border-color: #333;
}

/* Estilo para el botón negro */
.btn-black {
  background-color: black;
  color: white;
  font-weight: bold;
  padding: 6px 16px;
  border-radius: 5px;
  cursor: pointer;
  text-align: center;
  font-size: 0.875rem;
  transition: background-color 0.3s;
  border: none;
}

.btn-black:hover {
  background-color: #333;
}

/* Estilo para el input de archivo personalizado */
.custom-file-input-wrapper {
  position: relative;
  display: flex;
  align-items: center;
  gap: 8px;
}

.custom-file-input {
  display: none;
}

/* Estilo para botón de archivo */
.file-btn {
  background-color: #000;
  color: #fff;
  font-weight: bold;
  padding: 4px 8px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 0.875rem;
}

.file-btn:hover {
  background-color: #333;
}

.file-selected {
  font-size: 0.85rem;
  color: #555;
  margin-left: 8px;
}

/* Contenedor con scroll para los estampados */
.scrollable-container {
  overflow-y: auto;
  max-height: 400px;
  padding-right: 16px;
}

/* Espaciado y flexibilidad */
.flex {
  display: flex;
}

.flex-wrap {
  flex-wrap: wrap;
}

.gap-2 {
  gap: 8px;
}

/* Estilos del footer */
.footer {
  width: 100%;
  background-color: #333;
  color: #fff;
  padding: 20px;
  display: flex;
  justify-content: center;
  gap: 20px;
}

.footer a {
  color: #fff;
  text-decoration: none;
  font-weight: bold;
}

.footer a:hover {
  text-decoration: underline;
}

/* Estilos para las imágenes del carrusel */
.carousel-container .va-image {
  width: 100%;
  height: auto;
  max-height: 300px; /* Ajusta la altura máxima según tu necesidad */
  object-fit: contain; /* Ajuste para que las imágenes no se recorten */
}

/* Otros estilos adicionales */
</style>
