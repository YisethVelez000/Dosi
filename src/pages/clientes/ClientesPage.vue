<script setup lang="ts">
import UsersTable from '../clientes/widgets/ClientesTable.vue'
import { Cliente } from '../clientes/types'
import { useCliente } from '../clientes/composables/useClientes'
import router from '../../router'
import { removeCliente } from '../../data/pages/clientes'
const { clientes, isLoading, filters, sorting, pagination } = useCliente()

// Función para agregar un cliente
const AgregarCliente = () => {
  router.push('/clientes/agregar')
}

// Función para editar un cliente
const EditarCliente = (cliente: Cliente) => {
  router.push({
    path: `/clientes/editar/${cliente.IdCliente}`,
    query: { id: cliente.IdCliente },
  })
  console.log('Editar Cliente', cliente)
}

// Función para eliminar un cliente
const onUserDelete = (cliente: Cliente) => {
  removeCliente(cliente)
}

// Función para ver el detalle de un cliente
const VerDetalleCliente = (cliente: Cliente) => {
  router.push({
    path: `/clientes/detalle/${cliente.IdCliente}`,
    query: { id: cliente.IdCliente },
  })
}
</script>

<template>
  <h1 class="page-title">Clientes</h1>
  <VaCard>
    <VaCardContent>
      <div class="flex flex-col md:flex-row gap-2 mb-2 justify-between">
        <div class="flex flex-col md:flex-row gap-2 justify-start">
          <VaButtonToggle
            v-model="filters.isActive"
            color="background-element"
            border-color="background-element"
            :options="[
              { label: 'Activo', value: 'Activo' },
              { label: 'Inactivo', value: 'Inactivo' },
            ]"
          />
          <VaInput v-model="filters.search" placeholder="Buscar">
            <template #prependInner>
              <VaIcon name="search" color="secondary" size="small" />
            </template>
          </VaInput>
        </div>
        <VaButton @click="AgregarCliente">Añadir</VaButton>
      </div>

      <UsersTable
        v-model:sort-by="sorting.sortBy"
        v-model:sorting-order="sorting.sortingOrder"
        :clientes="clientes"
        :loading="isLoading"
        :pagination="pagination"
        @editUser="EditarCliente"
        @viewDetail="VerDetalleCliente"
        @deleteUser="onUserDelete"
      />
    </VaCardContent>
  </VaCard>
</template>
