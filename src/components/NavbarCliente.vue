<template>
  <nav class="navbar">
    <div class="navbar-logo">
      <img src="./../../public/logo de dosi.jpg" alt="Logo del software" class="logo" />
    </div>
    <div class="navbar-actions">
      <button v-if="!isLoggedIn" class="login-button" @click="irAllogin">Iniciar sesión</button>
      <div v-else class="user-icon">
        <div class="app-navbar-actions">
          <!-- Mostrar el nombre del usuario -->
          <span v-if="NombreUsuario">{{ NombreUsuario }}</span>
          <ProfileDropdown class="app-navbar-actions__item app-navbar-actions__item--profile mr-1" />
        </div>
      </div>
      <button class="carrito-button" @click="toggleCart">
        <img
          src="https://cdn.icon-icons.com/icons2/933/PNG/512/shopping-cart_icon-icons.com_72552.png"
          alt="Carrito"
          class="cart-icon"
        />
      </button>
    </div>
  </nav>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { jwtDecode } from 'jwt-decode'
import ProfileDropdown from '../components/navbar/components/dropdowns/ProfileDropdown.vue' // Importa el dropdown

const token = localStorage.getItem('token')

// Variable reactiva para el nombre de usuario
let NombreUsuario = ''

// Si el token existe, lo decodificamos y guardamos el nombre del usuario
if (token) {
  const decoded = jwtDecode(token) as any
  NombreUsuario = decoded.Nombre || ''
  localStorage.setItem('NombreUsuario', NombreUsuario)
  console.log('NombreUsuario', NombreUsuario, decoded)
}

// Emite eventos para que el componente padre los maneje
const emit = defineEmits(['toggleCart', 'irAllogin'])

// Funciones para emitir los eventos
const toggleCart = () => emit('toggleCart')
const irAllogin = () => emit('irAllogin')

// Variable reactiva que indica si el usuario está logueado
const isLoggedIn = ref(false)

// Verificar si existe un token en el localStorage al montar el componente
onMounted(() => {
  const token = localStorage.getItem('token')
  isLoggedIn.value = !!token // Si el token existe, isLoggedIn será true
})
</script>

<style scoped>
/* Aquí los estilos de tu navbar */
.navbar {
  width: 100%;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 20px;
  background-color: #ffffff;
  color: #333;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.navbar-logo {
  display: flex;
  justify-content: flex-start;
}

.logo {
  width: 80px; /* Ajusta este tamaño para hacer el logo más grande */
  height: auto;
}

.navbar-actions {
  display: flex;
  justify-content: flex-end;
}

.login-button {
  background: none;
  border: none;
  color: #000000;
  font-size: 1rem;
  cursor: pointer;
  margin-right: 20px;
}

.carrito-button {
  background: none;
  border: none;
  color: #000;
  font-size: 1.5rem;
  cursor: pointer;
  display: flex;
  align-items: center;
}

.cart-icon {
  width: 24px;
  height: 24px;
}

.person-icon {
  width: 24px;
  height: 24px;
  margin-right: 20px;
}
.app-navbar-actions {
  display: flex;
  align-items: center;

  .va-dropdown__anchor {
    color: var(--va-primary);
    fill: var(--va-primary);
  }

  &__item {
    padding: 0;
    margin-left: 0.25rem;
    margin-right: 0.25rem;

    svg {
      height: 20px;
    }

    &--profile {
      display: flex;
      justify-content: center;
    }

    .va-dropdown-content {
      background-color: var(--va-white);
    }

    @media screen and (max-width: 640px) {
      margin-left: 0;
      margin-right: 0;

      &:first-of-type {
        margin-left: 0;
      }
    }
  }

  .fa-github {
    color: var(--va-on-background-primary);
  }
}
</style>
