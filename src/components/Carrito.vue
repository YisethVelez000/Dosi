<script setup lang="ts">
import { ref, onMounted, defineExpose, computed } from 'vue'
import axios from 'axios'
import { jwtDecode } from 'jwt-decode'
const isCartOpen = ref(false)
const cartProducts = ref([])
const isUserRegistered = ref(false) // Nueva variable para verificar si el usuario está registrado
//Traemos el token
const token = localStorage.getItem('token')

let IdUsuario = ''
if (token) {
  const decoded = jwtDecode(token) as any
  IdUsuario = decoded.idUsuario || ''
  localStorage.setItem('IdUsuario', IdUsuario)
  console.log(IdUsuario)
  isUserRegistered.value = true
}
// Cargar productos del carrito desde localStorage al montar el componente
onMounted(() => {
  cartProducts.value = loadCartFromLocalStorage()
})

// Función para alternar el sidebar del carrito
const toggleCartSidebar = () => {
  isCartOpen.value = !isCartOpen.value
}

// Función para cargar productos desde localStorage
const loadCartFromLocalStorage = () => {
  const savedCart = localStorage.getItem('cart')
  return savedCart ? JSON.parse(savedCart) : []
}

// Función para guardar productos en localStorage
const saveCartToLocalStorage = (cartProducts) => {
  localStorage.setItem('cart', JSON.stringify(cartProducts))
}

// Función para eliminar un producto del carrito
const eliminarDelCarrito = (index) => {
  cartProducts.value.splice(index, 1)
  saveCartToLocalStorage(cartProducts.value)
}

// Función para incrementar la cantidad de un producto
const incrementQuantity = (index) => {
  cartProducts.value[index].Cantidad++
  updateProductInCart(cartProducts.value[index])
}

// Función para decrementar la cantidad de un producto
const decrementQuantity = (index) => {
  if (cartProducts.value[index].Cantidad > 1) {
    cartProducts.value[index].Cantidad--
    updateProductInCart(cartProducts.value[index])
  }
}

// Función para actualizar el producto en el backend o localmente
const updateProductInCart = async (product) => {
  console.log('Entrando ')
  if (isUserRegistered.value) {
    // Usuario registrado: actualizar en el backend
    try {
      console.log(IdUsuario)
      const response = await axios.put(
        `https://apinodedvp.onrender.com/carrito/carrito/${IdUsuario}/${product.IdProducto}`,
        {
          cantidad: product.Cantidad,
          IdUsuario: IdUsuario,
          IdProducto: product.IdProducto,
        },
      )

      if (response.status === 200) {
        console.log('Producto actualizado correctamente en el backend', response.data)
      }
    } catch (error) {
      console.error('Error al actualizar el producto en el carrito:', error)
    }
  } else {
    // Usuario no registrado: actualizar solo localStorage
    console.log('Usuario no registrado, actualizando carrito localmente.')
    saveCartToLocalStorage(cartProducts.value)
  }
}

// Función para calcular el total de los productos en el carrito
const totalCarrito = computed(() => {
  return cartProducts.value.reduce((total, product) => {
    return total + product.PrecioUnitario * product.Cantidad
  }, 0)
})

// Función para realizar el pedido
const realizarPedido = () => {
  console.log('Realizar pedido')
}

// Método para actualizar los productos en el carrito
const updateCartProducts = (products) => {
  cartProducts.value = products
  saveCartToLocalStorage(cartProducts.value)
}

// Expone los métodos al componente padre
defineExpose({
  toggleCartSidebar,
  updateCartProducts,
})
</script>

<template>
  <!-- Sidebar del carrito -->
  <div :class="['cart-sidebar', { 'is-open': isCartOpen }]">
    <div class="cart-sidebar-content">
      <button class="close-button" @click="toggleCartSidebar">X</button>
      <h3 class="sidebar-title">Carrito de Compras</h3>

      <!-- Aquí comienza la corrección -->
      <div v-if="cartProducts.length > 0" class="cart-items-container">
        <div v-for="(product, index) in cartProducts" :key="index" class="cart-item">
          <img :src="product.ImagenProducto" alt="Imagen del producto" class="cart-item-image" />
          <div class="cart-item-info">
            <h4>{{ product.NombreProducto }}</h4>
            <p>{{ product.PrecioUnitario }} x {{ product.Cantidad }}</p>
            <div class="quantity-control">
              <button class="quantity-button" @click="decrementQuantity(index)">-</button>
              <input
                v-model="product.Cantidad"
                type="number"
                min="1"
                class="quantity-input"
                @input="updateProductInCart(product)"
              />
              <button class="quantity-button" @click="incrementQuantity(index)">+</button>
            </div>
            <p>Total: {{ product.PrecioTotal }}</p>
            <button class="delete-button" @click="eliminarDelCarrito(index)">
              <!-- Icono de basura -->
              <svg
                xmlns="http://www.w3.org/2000/svg"
                width="16"
                height="16"
                fill="currentColor"
                class="bi bi-trash"
                viewBox="0 0 16 16"
              >
                <path
                  d="M5.5 5.5v6a.5.5 0 0 0 1 0v-6a.5.5 0 0 0-1 0zM3.5 5.5v6a.5.5 0 0 0 1 0v-6a.5.5 0 0 0-1 0zm6-2v10a.5.5 0 0 0 1 0V3.5a.5.5 0 0 0-1 0zM2.5 1a.5.5 0 0 0-.5.5v1a.5.5 0 0 0 .5.5H3v10a2 2 0 0 0 2 2h6a2 2 0 0 0 2-2V3h.5a.5.5 0 0 0 .5-.5v-1a.5.5 0 0 0-.5-.5H10v-.5a1 1 0 0 0-1-1H7a1 1 0 0 0-1 1V1H2.5zm3 0V1h4v1H5.5z"
                />
              </svg>
            </button>
          </div>
        </div>
        <!-- Mostrar total del carrito -->
        <div class="cart-total">
          <h4>Total del Carrito: {{ totalCarrito }}</h4>
        </div>
        <!-- Botón "Realizar Pedido" -->
        <button class="save-button realizar-pedido-button" @click="realizarPedido">Realizar Pedido</button>
      </div>
      <!-- Mover el bloque v-else directamente después del v-if -->
      <div v-else>
        <p>El carrito está vacío</p>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* Estilos del sidebar del carrito */
.cart-sidebar {
  position: fixed;
  top: 0;
  right: -300px;
  width: 300px;
  height: 100%;
  background-color: white;
  box-shadow: -2px 0 5px rgba(0, 0, 0, 0.2);
  overflow-y: auto; /* Mueve overflow-y aquí para el desplazamiento del sidebar */
  transition: right 0.3s ease-in-out;
  z-index: 1000;
}

.cart-sidebar.is-open {
  right: 0;
}

.cart-sidebar-content {
  padding: 20px;
  display: flex;
  flex-direction: column;
  height: 100%; /* Ajusta la altura para hacer el contenido más manejable */
}

.cart-items-container {
  flex-grow: 1; /* Permite que esta sección crezca y ocupe todo el espacio disponible */
  overflow-y: auto; /* Añade desplazamiento vertical */
  max-height: calc(100% - 150px); /* Ajusta el tamaño máximo para dejar espacio para otros elementos */
}

.close-button {
  position: absolute;
  top: 10px;
  right: 10px;
  background-color: transparent;
  border: none;
  font-size: 18px;
  cursor: pointer;
}

.sidebar-title {
  margin-bottom: 20px;
  font-size: 18px;
  font-weight: bold;
}

.cart-item {
  display: flex;
  align-items: center;
  margin-bottom: 20px;
  position: relative;
}

.cart-item-image {
  width: 50px;
  height: 50px;
  object-fit: cover;
  margin-right: 10px;
}

.cart-item-info {
  flex: 1;
  position: relative;
}

.cart-item-info button {
  margin-top: 10px;
  cursor: pointer;
}

.delete-button {
  background: none;
  border: none;
  cursor: pointer;
  margin-top: 10px;
}

.quantity-control {
  display: flex;
  align-items: center;
  gap: 5px;
  margin-top: 10px;
}

.quantity-button {
  width: 30px;
  height: 30px;
  display: flex;
  justify-content: center;
  align-items: center;
  border: 1px solid #ddd;
  background-color: #f9f9f9;
  cursor: pointer;
  font-size: 16px;
}

.quantity-input {
  width: 50px;
  text-align: center;
  margin: 0 10px;
  border: 1px solid #ddd;
}

/* Estilos del total del carrito */
.cart-total {
  margin-top: 20px;
  font-size: 18px;
  font-weight: bold;
  text-align: center;
}

/* Estilos del botón "Realizar Pedido" */
.realizar-pedido-button {
  display: block;
  width: 100%;
  padding: 10px;
  background-color: #000;
  color: #fff;
  text-align: center;
  font-weight: bold;
  border: none;
  cursor: pointer;
  margin-top: 20px;
  transition: background-color 0.3s ease;
}

.realizar-pedido-button:hover {
  background-color: #333;
}
</style>
