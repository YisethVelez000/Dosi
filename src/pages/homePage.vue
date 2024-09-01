<template>
  <div class="home-page">
    <!-- Usar el componente NavbarCliente -->
    <NavbarCliente @toggleCart="toggleCart" @irAllogin="irAllogin" />

    <!-- Componente de carrito -->
    <Carrito ref="carrito" />

    <!-- Sección de Servicios -->
    <section class="services-section">
      <div class="services-content">
        <h1>DOSI</h1>
        <h3>Prendas, Estampación y Personalización</h3>
      </div>
    </section>

    <!-- Sección de Novedades con carrusel -->
    <section class="novedades-section">
      <VaCarousel v-model="value" :items="items" :ratio="16 / 12" stateful autoscroll />
    </section>

    <!-- Nueva sección 'Quiénes Somos' -->
    <section class="about-us-section">
      <div class="about-us-content">
        <h2>Quiénes Somos</h2>
        <p>
          En DOSI, nos apasiona crear productos únicos y personalizados que reflejen la identidad de nuestros clientes.
          Nos enorgullece utilizar materiales de alta calidad y técnicas de estampación avanzadas para garantizar
          resultados excepcionales. Nuestro compromiso es brindar un servicio al cliente de primera clase, asegurando
          que cada experiencia con nosotros sea memorable y satisfactoria.
        </p>
      </div>
    </section>

    <!-- Sección de productos -->
    <div class="page-container">
      <div class="background">
        <div class="content-container">
          <div class="catalog">
            <div class="product-grid">
              <div v-for="product in paginatedProducts" :key="product.IdProducto" class="product-item">
                <VaCard class="product-card">
                  <div class="image-container">
                    <img
                      :src="getProductImage(product)"
                      :alt="product.NombreProductoCatalogo"
                      class="product-image"
                      @mouseover="zoomIn"
                      @mouseleave="zoomOut"
                    />
                  </div>
                  <div class="product-info">
                    <h3>{{ product.NombreProductoCatalogo }}</h3>
                    <!-- Mostrar precio en formato COP -->
                    <p><strong>Precio: </strong>{{ formatPrice(product.PrecioProducto) }}</p>
                    <p v-if="product.Personalizable === 'Si'" class="personalizable-text">
                      El producto es personalizable
                    </p>
                  </div>
                  <!-- Reemplazar el ícono de editar por la imagen del carrito -->
                  <RouterLink :to="{ name: 'productoCliente', params: { IdProducto: product.IdProducto } }">
                    <img
                      src="https://cdn.icon-icons.com/icons2/933/PNG/512/shopping-cart_icon-icons.com_72552.png"
                      alt="Agregar al carrito"
                      class="cart-icon"
                    />
                  </RouterLink>
                </VaCard>
              </div>
            </div>
          </div>
          <!-- Botones de paginación -->
          <div class="pagination">
            <button :disabled="currentPage === 1" class="pagination-button" @click="prevPage">Anterior</button>
            <span>Página {{ currentPage }} de {{ totalPages }}</span>
            <button :disabled="currentPage === totalPages" class="pagination-button" @click="nextPage">
              Siguiente
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- Pie de página -->
    <footer class="footer">
      <RouterLink to="https://www.instagram.com/dlalys_dosii/">Contacto</RouterLink>
      <RouterLink to="/privacy">Política de privacidad</RouterLink>
      <h1>DevelopmentA'S, 2024</h1>
    </footer>
  </div>
</template>

<script>
import axios from 'axios'
import { VaCarousel, VaCard } from 'vuestic-ui'
import Carrito from '../components/Carrito.vue' // Importa el componente de carrito
import NavbarCliente from '../components/NavbarCliente.vue' // Importa el componente NavbarCliente

export default {
  components: {
    VaCarousel,
    VaCard,
    Carrito, // Registra el componente de carrito
    NavbarCliente, // Registra el componente NavbarCliente
  },
  data() {
    return {
      value: 0,
      items: [
        'https://images.pexels.com/photos/1937336/pexels-photo-1937336.jpeg',
        'https://images.pexels.com/photos/6461325/pexels-photo-6461325.jpeg?auto=compress&cs=tinysrgb&w=1260&h=750&dpr=2',
        'https://crehana-blog.imgix.net/media/filer_public/01/f5/01f55d5e-1f4e-475f-9e87-5b917ad371c4/vinilo_textil.jpg?auto=format&q=50',
        'https://clarapatternstudio.com/wp-content/uploads/2022/05/que-es-serigrafia.jpeg',
        'https://tse1.mm.bing.net/th?id=OIP.W_9Y8BMzTsauzJNdf-CllwHaCy&pid=Api&P=0&h=180',
      ],
      products: [],
      currentPage: 1,
      itemsPerPage: 20,
    }
  },
  computed: {
    filteredProducts() {
      return this.products.filter((product) => product.EstadoProducto === 'Activo')
    },
    paginatedProducts() {
      const start = (this.currentPage - 1) * this.itemsPerPage
      const end = start + this.itemsPerPage
      return this.filteredProducts.slice(start, end)
    },
    totalPages() {
      return Math.ceil(this.filteredProducts.length / this.itemsPerPage)
    },
  },
  created() {
    this.fetchProducts()
  },
  methods: {
    async fetchProducts() {
      try {
        const response = await axios.get('https://apinodedvp.onrender.com/productos')
        console.log('Productos recibidos:', response.data)
        this.products = response.data
      } catch (error) {
        console.error('Error al obtener productos:', error)
      }
    },
    getProductImage(product) {
      if (product.Producto_Has_Imagens && product.Producto_Has_Imagens.length > 0) {
        return product.Producto_Has_Imagens[0].Imagen.Imagen
      }
      return 'https://via.placeholder.com/300' // Imagen por defecto si no hay ninguna
    },
    zoomIn(event) {
      const image = event.target
      image.style.transform = 'scale(1.1)'
      image.style.transition = 'transform 0.3s ease-in-out'
    },
    zoomOut(event) {
      const image = event.target
      image.style.transform = 'scale(1)'
    },
    irAllogin() {
      this.$router.push({ name: 'login' })
    },
    toggleCart() {
      this.$refs.carrito.toggleCartSidebar() // Llama al método expuesto
    },
    prevPage() {
      if (this.currentPage > 1) {
        this.currentPage--
      }
    },
    nextPage() {
      if (this.currentPage < this.totalPages) {
        this.currentPage++
      }
    },
    formatPrice(value) {
      return value.toLocaleString('es-CO', { style: 'currency', currency: 'COP' })
    },
  },
}
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Merriweather:wght@300;400&display=swap');

.home-page {
  display: flex;
  flex-direction: column;
  align-items: center;
  background-color: #f7f7f7;
  color: #333;
  font-family: 'Open Sans', sans-serif;
  width: 100%;
  box-sizing: border-box;
}

.services-section {
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 40px 0 20px 0; /* Reducido para eliminar el espacio adicional */
  background-color: #3d385f; /* Fondo semi-transparente */
  background-size: cover;
  background-position: center;
  color: white;
  text-align: center;
}

.services-content {
  max-width: 800px;
  font-family: 'Merriweather', serif; /* Aplicando Merriweather */
}

.services-content h1 {
  font-size: 3rem;
  font-weight: 300; /* Usar Merriweather Light */
  margin-bottom: 20px;
}

.services-content h3 {
  font-size: 2rem;
  font-weight: 300; /* Usar Merriweather Light */
  margin-bottom: 20px;
}

.services-content p {
  font-size: 1.2rem;
}

.novedades-section {
  width: 100%;
  height: 500px;
  margin: 0 auto; /* Elimina espacio superior e inferior */
}

.about-us-section {
  width: 100%;
  padding: 40px 20px;
  background-color: beige;
  text-align: center;
  color: #333;

  font-family: 'Helvetica Neue', sans-serif; /* Manteniendo Helvetica Light */
}

.about-us-content {
  max-width: 800px;
  margin: 0 auto;
}

.about-us-content h2 {
  font-size: 2.5rem;
  font-weight: lighter; /* Usar Helvetica Light */
  margin-bottom: 20px;
}

.about-us-content p {
  font-size: 1.2rem;
  line-height: 1.5;
}

.page-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  background-image: url('https://img.freepik.com/foto-gratis/fondo_53876-32170.jpg?w=900&t=st=1723158687~exp=1723159287~hmac=ca7360f6388e32be58092afafa63e288d84316777a9651ab19c310a192076f95');
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
  background-attachment: fixed;
  padding: 20px;
  width: 100%;
}

.background {
  width: 100%;
  max-width: 1200px;
}

.content-container {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.catalog {
  width: 100%;
}

.product-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 10px;
  justify-content: center;
}

.product-item {
  margin: 10px;
  display: flex;
  flex-direction: column;
}

.product-card {
  background-color: rgba(232, 223, 223, 0.9);
  border-radius: 8px;
  overflow: hidden;
  position: relative;
  height: 100%;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

.image-container {
  width: 100%;
  padding-top: 100%;
  position: relative;
  overflow: hidden;
}

.product-image {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: contain;
  transition: transform 0.3s ease-in-out;
}

.cart-icon {
  position: absolute;
  bottom: 10px; /* Ajusta el espacio desde el fondo */
  left: 10px; /* Ajusta el espacio desde la izquierda */
  cursor: pointer;
  width: 24px; /* Ajusta el tamaño de la imagen */
  height: 24px; /* Ajusta el tamaño de la imagen */
}

.product-info {
  padding: 10px;
  text-align: center;
  flex-grow: 1;
  position: relative; /* Añadir esto para que el ícono del carrito se posicione relativo a este contenedor */
  margin-bottom: 30px; /* Añadir margen para evitar que el icono del carrito se superponga con el texto */
}

.product-info h3 {
  font-size: 1.2rem;
  margin: 0 0 5px;
  font-weight: bold;
  color: #333;
  font-family: 'Helvetica Neue', Arial, sans-serif;
}

.product-info p {
  margin: 0 0 5px;
  font-size: 1rem;
  color: #534da0;
  font-family: 'Helvetica Neue', Arial, sans-serif;
}

.edit-button {
  position: absolute;
  bottom: 10px;
  left: 10px;
  color: #000;
}

.footer {
  width: 100%;
  background-color: #3d385f;
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

.pagination {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-top: 20px;
}

.pagination button.pagination-button {
  background-color: #000;
  color: #fff;
  border: none;
  padding: 10px 20px;
  cursor: pointer;
  margin: 0 10px;
  font-weight: bold;
  transition: background-color 0.3s ease;
}

.pagination button.pagination-button:hover {
  background-color: #333;
}

.pagination button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

@media (max-width: 600px) {
  .novedades-section {
    width: 100%;
  }

  .product-grid {
    grid-template-columns: repeat(2, 1fr);
  }

  .about-us-content h2 {
    font-size: 2rem;
  }

  .about-us-content p {
    font-size: 1rem;
  }
}
</style>
