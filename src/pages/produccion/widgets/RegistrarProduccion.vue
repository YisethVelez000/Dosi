<script setup lang="ts">
import { onMounted, ref } from 'vue'
import { useRouter } from 'vue-router'
import { VaFile, VaFileUpload } from 'vuestic-ui/web-components'

// Usamos el router
const router = useRouter()

const productos = ref<
  {
    IdProducto: number
    NombreProductoCatalogo: string
    Producto_has_Colores: { IdColor: number; Color: string }[]
    Producto_has_Tallas: { IdTalla: number; Talla: string }[]
  }[]
>([])
const coloresDisponibles = ref<{ IdColor: number; Color: string }[]>([])
const tallasDisponibles = ref<{ IdTalla: number; Talla: string }[]>([])
const estampados = ref<{ IdEstampado: number; TipoEstampado: string }[]>([])

// Función para obtener los productos
const fetchProductos = async () => {
  try {
    const response = await fetch('https://apinodedvp.onrender.com/productos')
    if (response.ok) {
      productos.value = await response.json()
    } else {
      console.error('Error al obtener los productos')
    }
  } catch (error) {
    console.error('Error al obtener los productos:', error)
  }
}

// Función para obtener la lista de colores desde la API
const fetchColores = async () => {
  try {
    const response = await fetch('https://apinodedvp.onrender.com/colores/listarColores')
    if (response.ok) {
      const data = await response.json()
      // Acceder al campo 'data' y mapear los colores
      coloresDisponibles.value = data.data.map((color: { IdColor: number; Color: string }) => ({
        IdColor: color.IdColor,
        Color: color.Color,
      }))
    } else {
      console.error('Error al obtener los colores')
    }
  } catch (error) {
    console.error('Error al obtener los colores:', error)
  }
}

const fetchTallas = async () => {
  try {
    const response = await fetch('https://apinodedvp.onrender.com/talla/listarTalla')
    if (response.ok) {
      const data = await response.json()
      // Mapear las tallas como un array de objetos con IdTalla y Talla
      tallasDisponibles.value = data.map((talla: { IdTalla: number; Talla: string }) => ({
        IdTalla: talla.IdTalla,
        Talla: talla.Talla,
      }))
    } else {
      console.error('Error al obtener las tallas')
    }
  } catch (error) {
    console.error('Error al obtener las tallas:', error)
  }
}

const ubicacionesOptions = ref<{ IdUbicacion: number; Ubicacion: string }[]>([])

const fetchUbicaciones = async () => {
  try {
    const response = await fetch('https://apinodedvp.onrender.com/ubicaciones')
    if (response.ok) {
      const data = await response.json()
      // Asignamos el resultado al ref 'ubicacionesOptions'
      ubicacionesOptions.value = data.map((ubicacion: { IdUbicacion: number; Ubicacion: string }) => ({
        IdUbicacion: ubicacion.IdUbicacion,
        Ubicacion: ubicacion.Ubicacion,
      }))
    } else {
      console.error('Error al obtener las ubicaciones')
    }
  } catch (error) {
    console.error('Error al obtener las ubicaciones:', error)
  }
}

const fetchEstampados = async () => {
  try {
    const response = await fetch('https://apinodedvp.onrender.com/estampados')
    if (response.ok) {
      estampados.value = await response.json()
    } else {
      console.error('Error al obtener los estampados')
    }
  } catch (error) {
    console.error('Error al obtener los estampados:', error)
  }
}

const terciariasOptions = ref<{ IdEmpresa: number; NombreEmpresa: string }[]>([])

const fetchTerciarias = async () => {
  try {
    const response = await fetch('https://apinodedvp.onrender.com/terciarias')
    if (response.ok) {
      const data = await response.json()
      // Asignamos el resultado al ref 'terciariasOptions'
      terciariasOptions.value = data.map((empresa: { IdEmpresa: number; NombreEmpresa: string }) => ({
        IdEmpresa: empresa.IdEmpresa,
        NombreEmpresa: empresa.NombreEmpresa,
      }))
    } else {
      console.error('Error al obtener las empresas terciarias')
    }
  } catch (error) {
    console.error('Error al obtener las empresas terciarias:', error)
  }
}

const updateDisponibles = async (IdProducto: number) => {
  try {
    // Encuentra el producto seleccionado
    const productoSeleccionado = productos.value.find((producto) => producto.IdProducto === IdProducto)

    if (productoSeleccionado && productoSeleccionado.Producto_has_Colores) {
      // Extrae los IdColor asociados al producto
      const coloresAsociados = productoSeleccionado.Producto_has_Colores.map((relacion) => relacion.IdColor)
      console.log(coloresAsociados)
      // Filtra los colores disponibles que coinciden con los IdColor asociados
      coloresDisponibles.value = coloresDisponibles.value.filter((color) => coloresAsociados.includes(color.IdColor))
    } else {
      coloresDisponibles.value = []
    }

    //Hacemos lo mismo con las tallas
    if (productoSeleccionado && productoSeleccionado.Producto_has_Tallas) {
      const tallasAsociadas = productoSeleccionado.Producto_has_Tallas
      // Filtra las tallas disponibles que coinciden con las tallas asociadas
      console.log(tallasAsociadas)
      tallasDisponibles.value = tallasDisponibles.value.filter((talla) =>
        tallasAsociadas.map((t) => t.IdTalla).includes(talla.IdTalla),
      )
    } else {
      tallasDisponibles.value = []
    }
  } catch (error) {
    console.error('Error al actualizar colores disponibles:', error)
  }
}

const onProductoChangePersonalizado = () => {
  const productoSeleccionado = nuevoProductoPersonalizado.value?.IdProducto
  if (productoSeleccionado) {
    updateDisponibles(Number(productoSeleccionado))
  }
}

onMounted(() => {
  fetchProductos()
  fetchColores()
  fetchTallas()
  fetchUbicaciones()
  fetchEstampados()
  fetchTerciarias()
})
const nuevoProducto = ref({
  IdProducto: '',
  Color: '',
  Talla: '',
  Cantidad: 0,
})

const nuevoProductoPersonalizado = ref({
  IdProducto: '',
  Color: '',
  Talla: '',
  Cantidad: 0,
  imagen: undefined as VaFile | VaFile[] | undefined,
  IdUbicacion: 0,
  IdEstampado: 0,
})

const showModalEstampados = ref(false)
const showModal = ref(false)

const nuevaOrdenProduccion = ref({
  NumeroOrdenProduccion: 0,
  FechaEstimada: Date.toString(),
  EsProduccionInterna: '',
  EstadoOrden: '',
  Produccion_Has_Producto: [],
  Estampado: [],
  Terciarias: [
    {
      IdEmpresa: 0,
      ValorAPagar: 0,
    },
  ],
})

const productosList = ref<Array<{ IdProducto: string; Color: string; Talla: string; Cantidad: number }>>([])

const productosPersonalizados = ref<
  Array<{
    IdProducto: string
    Color: string
    Talla: string
    Cantidad: number
    imagen?: VaFile | VaFile[] | undefined
  }>
>([])

const removeProduct = (index: number, type: string): void => {
  if (type === 'producto') {
    productosList.value.splice(index, 1)
  } else {
    productosPersonalizados.value.splice(index, 1)
  }
}

const addProduct = () => {
  const producto = { ...nuevoProducto.value }
  productosList.value.push(producto)
  nuevoProducto.value = {
    IdProducto: '',
    Color: '',
    Talla: '',
    Cantidad: 0,
  }
}

const addProductPersonalizado = () => {
  const productoPersonalizado = { ...nuevoProductoPersonalizado.value }
  productosPersonalizados.value.push(productoPersonalizado)
  nuevoProductoPersonalizado.value = {
    IdProducto: '',
    Color: '',
    Talla: '',
    Cantidad: 0,
    imagen: undefined,
    IdUbicacion: 0,
    IdEstampado: 0,
  }
  showModalEstampados.value = false
}
const showModalImagen = ref(false)
const imagenSeleccionada = ref<string | undefined>()

const verImagen = (imagen: VaFile | VaFile[] | undefined) => {
  if (Array.isArray(imagen)) {
    imagenSeleccionada.value = imagen.length > 0 ? URL.createObjectURL(imagen[0]) : undefined
  } else if (imagen) {
    imagenSeleccionada.value = URL.createObjectURL(imagen)
  } else {
    imagenSeleccionada.value = undefined
  }
  showModalImagen.value = true
}

const cancelar = () => {
  router.push('/produccion')
}

// Función para agrupar productos por IdProducto y sumar las cantidades
const agruparProductosYSumarCantidades = (
  productos: Array<{ IdProducto: string; Color: string; Talla: string; Cantidad: number }>,
) => {
  const productosAgrupados: Record<string, { IdProducto: string; Color: string; Talla: string; Cantidad: number }> = {}

  productos.forEach((producto) => {
    const key = `${producto.IdProducto}-${producto.Color}-${producto.Talla}`
    if (productosAgrupados[key]) {
      // Si ya existe, suma la cantidad
      productosAgrupados[key].Cantidad += producto.Cantidad
    } else {
      // Si no existe, lo añade
      productosAgrupados[key] = { ...producto }
    }
  })

  // Convertir de objeto a array
  return Object.values(productosAgrupados)
}
//Creamos un array de imagenes donde asignamos la de los productos personalizados
const imagenes = ref<VaFile[]>([])
//Creamos un array de imagenes renombradas
const imagenesRenombradas = ref<string[]>([])
//Recorremos el array de imagenes de productos personalizados y lo asignamos a imagenes
for (let i = 0; i < productosPersonalizados.value.length; i++) {
  if (productosPersonalizados.value[i].imagen) {
    imagenes.value.push(productosPersonalizados.value[i].imagen)
  }
}

//Recorremos el array de imagenes y le asignamos un nombre
for (let i = 0; i < imagenes.value.length; i++) {
  imagenesRenombradas.value.push(`imagen${i}`)
}

//Renombramos el array productosPersonalizados y le quita el campo imagen
const Estampado = ref<Array<{ IdProducto: string; Color: string; Talla: string; Cantidad: number }>>([])
console.log(Estampado)

//Craeamos el onSave para guardar la orden de produccion
const onSave = () => {
  const formData = new FormData()
  // Agrupar productos y sumar cantidades
  const productosListAgrupados = agruparProductosYSumarCantidades(productosList.value)
  const productosPersonalizadosAgrupados = agruparProductosYSumarCantidades(
    productosPersonalizados.value.map((producto) => {
      const { imagen, ...productoSinImagen } = producto
      console.log('imagen:', imagen)
      return productoSinImagen
    }),
  )
  console.log('Productos Personalizados agrupados:', productosPersonalizadosAgrupados)
  console.log('Productos agrupados:', productosListAgrupados)
  // Añadir la fecha estimada, estado, y si es producción interna
  formData.append('FechaEstimada', nuevaOrdenProduccion.value.FechaEstimada)
  formData.append('EsProduccionInterna', nuevaOrdenProduccion.value.EsProduccionInterna)
  formData.append('EstadoOrden', nuevaOrdenProduccion.value.EstadoOrden)

  // Añadir las empresas terciarias con el valor a pagar
  formData.append('Terciarias', JSON.stringify(nuevaOrdenProduccion.value.Terciarias))

  // Añadir los productos sin imagen
  formData.append('Produccion_Has_Producto', JSON.stringify(productosList.value))

  // Procesar productos personalizados quitando la imagen y verificando IdProducto
  const productosPersonalizadosSinImagen = productosPersonalizados.value.map((producto) => {
    const { imagen, ...productoSinImagen } = producto // Desestructuración para eliminar imagen
    console.log(imagen)
    return productoSinImagen
  })
  formData.append('Estampado', JSON.stringify(productosPersonalizadosSinImagen))

  // Añadir las imágenes de los productos personalizados
  imagenes.value.forEach((imagen, index) => {
    const nombreImagen = `imagen${index}` // Nombre único para cada imagen
    const blob = new Blob([imagen], { type: imagen.type })
    formData.append(nombreImagen, blob, imagen.name)
  })

  // Imprimir para verificar el contenido del formData (opcional)
  formData.forEach((value, key) => {
    console.log(key, value)
  })
  //Hacemos una funcion para sumar la cantidad de todos los productos con ids iguales y sumamos la cantidad

  // Realizar la petición
  fetch('https://apinodedvp.onrender.com/ordenesProduccion', {
    method: 'POST',
    body: formData,
  })
    .then((response) => {
      if (response.ok) {
        // Aquí haces el GET al API de productos
        //Recorremos el array de productosListAgrupados
        for (let i = 0; i < productosListAgrupados.length; i++) {
          //Hacemos un fetch para obtener el producto por id
          fetch(`https://apinodedvp.onrender.com/productos/${productosListAgrupados[i].IdProducto}`)
            .then((productResponse) => productResponse.json())
            .then((product) => {
              // Sumamos la cantidad al stock
              const cantidad = productosListAgrupados[i].Cantidad
              const nuevoStock = product.Stock + cantidad
              console.log('Nuevo stock:', nuevoStock)
              // Hacemos un PUT para actualizar el stock
              return fetch(`https://apinodedvp.onrender.com/productos/${productosListAgrupados[i].IdProducto}`, {
                method: 'PUT',
                headers: {
                  'Content-Type': 'application/json',
                },
                body: JSON.stringify({ Stock: nuevoStock }),
              })
            })
            .then((updateResponse) => {
              if (updateResponse.ok) {
                console.log('Stock actualizado correctamente')
                router.push('/produccion')
              } else {
                console.error('Error al actualizar el stock')
              }
            })
            .catch((error) => {
              console.error('Error al obtener o actualizar el producto:', error)
            })
        }
      } else {
        console.error('Error al guardar la orden de producción')
      }
    })
    .catch((error) => {
      console.error('Error al guardar la orden de producción:', error)
    })
}
</script>
<template>
  <div>
    <h2 class="text-xl font-semibold mb-4">Agregar</h2>
    <VaCard>
      <VaCardContent>
        <div class="self-stretch flex-col justify-start items-start gap-4 flex">
          <div class="flex gap-4 flex-col sm:flex-row w-full">
            <VaInput
              v-model="nuevaOrdenProduccion.FechaEstimada"
              class="w-full sm:w-1/2"
              name="FechaEstimada"
              label="Fecha Estimada"
              type="date"
            />
            <VaSelect
              v-model="nuevaOrdenProduccion.EsProduccionInterna"
              class="w-full sm:w-1/2"
              name="EsProduccionInterna"
              label="Producción Interna"
              :options="['SI', 'NO']"
            />
          </div>
          <div class="flex gap-4 flex-col sm:flex-row w-full">
            <VaSelect
              v-model="nuevaOrdenProduccion.EstadoOrden"
              class="w-full sm:w-1/2"
              name="EstadoOrden"
              label="Estado"
              :options="['Pendiente', 'Iniciado', 'Estampando', 'Finalizado', 'En Tercera']"
            />
          </div>

          <div class="flex gap-4 flex-col sm:flex-row w-full">
            <div class="form-group">
              <label for="empresaTerciaria" class="form-label">Empresa Terciaria</label>
              <select id="empresaTerciaria" v-model="nuevaOrdenProduccion.Terciarias[0].IdEmpresa" class="form-control">
                <option value="" disabled>Seleccione una empresa terciaria</option>
                <option v-for="empresa in terciariasOptions" :key="empresa.IdEmpresa" :value="empresa.IdEmpresa">
                  {{ empresa.NombreEmpresa }}
                </option>
              </select>
            </div>
            <VaInput
              v-model="nuevaOrdenProduccion.Terciarias[0].ValorAPagar"
              class="w-full sm:w-1/2"
              name="ValorAPagar"
              label="Valor a Pagar"
              type="number"
            />
          </div>
          <div class="flex gap-4 flex-col sm:flex-row w-full mt-4">
            <div class="w-full sm:w-1/2">
              <!-- Tabla para Productos -->
              <div class="w-full">
                <h1 class="text-xl font-semibold mb-4">
                  Listado Productos
                  <VaButton class="ml-4" size="small" icon="add" @click="showModal = true" />
                </h1>
                <table class="styled-table mt-4">
                  <thead>
                    <tr>
                      <th>Nombre Producto</th>
                      <th>Color</th>
                      <th>Talla</th>
                      <th>Cantidad</th>
                      <th>Acciones</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="(producto, index) in productosList" :key="index">
                      <td>{{ producto.IdProducto }}</td>
                      <td>{{ producto.Color }}</td>
                      <td>{{ producto.Talla }}</td>
                      <td>{{ producto.Cantidad }}</td>
                      <td>
                        <VaButton
                          preset="primary"
                          size="small"
                          icon="mso-delete"
                          color="danger"
                          aria-label="Eliminar Producto"
                          title="Eliminar"
                          @click="removeProduct(index, 'producto')"
                        />
                      </td>
                    </tr>
                  </tbody>
                </table>
              </div>
            </div>
            <div class="w-full sm:w-1/2">
              <!-- Tabla para Productos Personalizados -->
              <div class="w-full mt-">
                <h1 class="text-xl font-semibold mb-4">
                  Listado Productos Personalizados
                  <VaButton class="ml-4" size="small" icon="add" @click="showModalEstampados = true" />
                </h1>
                <table class="styled-table">
                  <thead>
                    <tr>
                      <th>Nombre Producto</th>
                      <th>Color</th>
                      <th>Talla</th>
                      <th>Cantidad</th>
                      <th>Acciones</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="(producto, index) in productosPersonalizados" :key="index">
                      <td>{{ producto.IdProducto }}</td>
                      <td>{{ producto.Color }}</td>
                      <td>{{ producto.Talla }}</td>
                      <td>{{ producto.Cantidad }}</td>
                      <td>
                        <VaButton
                          class="mr-5"
                          size="small"
                          icon="visibility"
                          aria-label="Ver Imagen"
                          title="Ver Imagen"
                          @click="verImagen(producto.imagen)"
                        />

                        <VaButton
                          preset="primary"
                          size="small"
                          icon="mso-delete"
                          color="danger"
                          aria-label="Eliminar Producto"
                          title="Eliminar"
                          @click="removeProduct(index, 'personalizado')"
                        />
                      </td>
                    </tr>
                  </tbody>
                </table>
              </div>
            </div>
          </div>
        </div>
        <div class="flex justify-end space-x-4 mt-4">
          <VaButton preset="secondary" color="secondary" @click="cancelar">Cancelar</VaButton>
          <VaButton preset="primary" color="primary" @click="onSave">Guardar</VaButton>
        </div>
      </VaCardContent>
    </VaCard>
    <!-- Modal para agregar productos -->
    <VaModal
      v-model="showModal"
      title="Agregar Nuevo Producto"
      :cancel-text="'Cancelar'"
      :ok-text="'Agregar'"
      @ok="addProduct"
    >
      <template #default>
        <div class="form-group">
          <label for="producto" class="form-label">NOMBRE PRODUCTO</label>
          <select id="producto" v-model="nuevoProducto.IdProducto" class="form-control">
            <option value="" disabled>Seleccione un producto</option>
            <option v-for="producto in productos" :key="producto.IdProducto" :value="producto.IdProducto">
              {{ producto.IdProducto }}
            </option>
          </select>
        </div>
        <!-- Otros campos para el producto -->
        <VaInput v-model="nuevoProducto.Color" class="w-full" name="Color" label="Color" type="text" />
        <VaInput v-model="nuevoProducto.Talla" class="w-full" name="Talla" label="Talla" type="text" />
        <VaInput v-model="nuevoProducto.Cantidad" class="w-full" name="Cantidad" label="Cantidad" type="number" />
      </template>
    </VaModal>

    <!-- Modal para agregar productos personalizados -->
    <VaModal
      v-model="showModalEstampados"
      title="Agregar Producto Personalizado"
      :cancel-text="'Cancelar'"
      :ok-text="'Agregar'"
      @ok="addProductPersonalizado"
    >
      <template #default>
        <div class="form-group">
          <label for="productoPersonalizado" class="form-label">NOMBRE PRODUCTO</label>
          <select
            id="productoPersonalizado"
            v-model="nuevoProductoPersonalizado.IdProducto"
            class="form-control"
            @change="onProductoChangePersonalizado"
          >
            <option value="" disabled>Seleccione un producto</option>
            <option v-for="producto in productos" :key="producto.IdProducto" :value="producto.IdProducto">
              {{ producto.NombreProductoCatalogo }}
            </option>
          </select>
        </div>
        <div class="form-group">
          <label for="colorPersonalizado" class="form-label">Color</label>
          <select id="colorPersonalizado" v-model="nuevoProductoPersonalizado.Color" class="form-control">
            <option value="" disabled>Seleccione un color</option>
            <option v-for="color in coloresDisponibles" :key="color.IdColor" :value="color.IdColor">
              {{ color.Color }}
            </option>
          </select>
        </div>
        <div class="form-group">
          <label for="tallaPersonalizado" class="form-label">Talla</label>
          <select id="tallaPersonalizado" v-model="nuevoProductoPersonalizado.Talla" class="form-control">
            <option value="" disabled>Seleccione una talla</option>
            <option v-for="talla in tallasDisponibles" :key="talla.IdTalla" :value="talla.IdTalla">
              {{ talla.Talla }}
            </option>
          </select>
        </div>
        <VaInput v-model="nuevoProductoPersonalizado.Cantidad" class="w-full" name="Cantidad" label="Cantidad" />
        <!-- Ubicación con el fetch  -->
        <div class="form-group">
          <label for="ubicacion" class="form-label">Ubicación</label>
          <select id="ubicacion" v-model="nuevoProductoPersonalizado.IdUbicacion" class="form-control">
            <option value="" disabled>Seleccione una ubicación</option>
            <option v-for="ubicacion in ubicacionesOptions" :key="ubicacion.IdUbicacion" :value="ubicacion.IdUbicacion">
              {{ ubicacion.Ubicacion }}
            </option>
          </select>
        </div>
        <VaFileUpload
          v-model="nuevoProductoPersonalizado.imagen"
          upload-button-tex="'Subir Archivos'"
          type="single"
          class="bg-primary"
        >
          <div class="custom-upload-file-area">
            <p class="mb-2">Haz clic o arrastra y suelta archivos aquí para subir</p>
            <!-- El input tipo file generado por VaFileUpload -->
            <input type="file" accept="image/*" class="custom-file-input" />
          </div>
        </VaFileUpload>
        <!-- Estampado con el fetch -->
        <div class="form-group">
          <label for="estampado" class="form-label">Estampado</label>
          <select id="estampado" v-model="nuevoProductoPersonalizado.IdEstampado" class="form-control">
            <option value="" disabled>Seleccione un estampado</option>
            <option v-for="estampado in estampados" :key="estampado.IdEstampado" :value="estampado.IdEstampado">
              {{ estampado.TipoEstampado }}
            </option>
          </select>
        </div>
      </template>
    </VaModal>
    <!-- Modal para ver la imagen del producto personalizado -->
    <VaModal v-model="showModalImagen" title="Ver Imagen" :cancel-text="'Cerrar'" @cancel="showModalImagen = false">
      <template #default>
        <div v-if="imagenSeleccionada">
          <img :src="imagenSeleccionada" alt="Imagen del Producto Personalizado" class="w-full h-auto" />
        </div>
        <div v-else>
          <p>No hay imagen disponible</p>
        </div>
      </template>
    </VaModal>
  </div>
</template>

<style scoped>
/* Estilos para las tablas */
.v-modal img {
  max-width: auto;
  height: auto;
}
.styled-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 16px;
  font-family: Arial, sans-serif;
  margin: 10px, 0, 0, 0;
}

.styled-table td {
  padding: 5px 0px;
}

.styled-table thead th {
  color: #000000;
  text-align: left;
  /* Cambia esto a 'center' si quieres centrar el texto */
  padding: 0px;
  /* Ajusta el padding para reducir espacio */
}

.form-group {
  margin-bottom: 1rem;
}

.form-label {
  display: block;
  color: #154ec1;
  /* Azul */
  font-size: 9px;
  line-height: 14px;
  letter-spacing: 0.4px;
  min-height: 14px;
  font-weight: bold;
}

.form-control {
  display: block;
  width: 100%;
  padding: 0.5rem 1rem;
  font-size: 1rem;
  line-height: 1.5;
  color: #495057;
  background-color: #fff;
  background-clip: padding-box;
  border: 1px solid #ced4da;
  border-radius: 0.25rem;
  transition:
    border-color 0.15s ease-in-out,
    box-shadow 0.15s ease-in-out;
}

.form-control:focus {
  border-color: #2563eb;
  outline: 0;
  box-shadow: 0 0 0 0.2rem rgba(37, 99, 235, 0.25);
}

.va-select.va-input {
  background-color: #f9fafb;
  border-radius: 8px;
}
</style>
