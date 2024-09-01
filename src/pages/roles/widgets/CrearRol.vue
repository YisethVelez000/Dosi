<template>
  <h2 class="text-xl font-semibold mb-4">Agregar Rol</h2>
  <VaCard>
    <VaCardContent>
      <div class="flex flex-col gap-4 flex-col sm:flex-row w-full">
        <VaInput
          v-model="role.nombre"
          label="Nombre"
          placeholder="Ingresa el nombre del rol"
          class="w-full"
          :rules="[
            (v: any) => !!v || 'El nombre es requerido',
            (v: string | any[]) => (v && v.length <= 50) || 'El nombre no puede tener más de 50 caracteres',
            (v: string | any[]) => (v && v.length >= 3) || 'El nombre no puede tener menos de 3 caracteres',
            //Validamos que no existan espacio en blanco
            (v: string) => v.trim().length > 0 || 'El nombre no puede contener solo espacios en blanco',
          ]"
        />
        <VaTextarea
          v-model="role.descripcion"
          label="Descripción"
          placeholder="Ingresa la descripción del rol"
          class="w-full"
          :rows="4"
          :rules="[
            (v: any) => !!v || 'La descripción es requerida',
            (v: string | any[]) => (v && v.length >= 3) || 'La descripción no puede tener menos de 3 caracteres',
            // Validamos que no existan espacios en blanco
            (v: string) => v.trim().length > 0 || 'La descripción no puede contener solo espacios en blanco',
          ]"
        />
      </div>
      <div class="table-container">
        <table class="table-auto w-full">
          <thead class="bg-gray-200">
            <tr>
              <th class="px-4 py-2 text-center">Módulo</th>
              <th v-for="permiso in permisos" :key="permiso" class="px-4 py-2 text-center">{{ permiso }}</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="modulo in role.modulos" :key="modulo.nombre">
              <td class="border px-4 py-2 text-center">{{ modulo.nombre }}</td>
              <td v-for="permiso in permisos" :key="permiso" class="border px-4 py-2 text-center">
                <div class="flex items-center justify-center">
                  <input
                    :id="modulo.nombre + '-' + permiso"
                    v-model="modulo.permisosSeleccionados"
                    type="checkbox"
                    :value="permiso"
                    class="h-4 w-4 text-indigo-600 border-gray-300 rounded focus:ring-indigo-500"
                  />
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
      <div class="flex justify-end space-x-4 mt-4">
        <VaButton preset="secondary" color="secondary" @click="cancelar">Cancelar</VaButton>
        <VaButton @click="guardar">Guardar</VaButton>
      </div>
    </VaCardContent>
  </VaCard>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { VaCard, VaCardContent, VaInput, VaButton } from 'vuestic-ui'
import { addRol } from '../../../data/pages/roles'
import { useRouter } from 'vue-router'

// Usamos el router
const router = useRouter()
// Usamos el mismo mapa en ambos componentes
const modulosMap = {
  'Ventas-vista-cliente': 1,
  'Ventas-vista-administrador': 2,
  Insumos: 3,
  Pedidos: 4,
  'productos-vista-cliente': 5,
  Compras: 6,
  parametrización: 7,
  'ubicaciones-vista-administrador': 8,
  'ubicaciones-vista-cliente': 9,
  'estampados-vista-administrador': 10,
  'estampados-vista-cliente': 11,
  'productos-vista-administrador': 12,
  'Ficha tecnica': 14,
  Colores: 15,
  'Empresas Terceras': 16,
  Proveedores: 17,
  Clientes: 18,
  Roles: 19,
  Descuentos: 20,
  'Orden Produccion': 21,
  Usuarios: 22,
}

const permisos = ['Crear', 'Ver', 'Editar', 'Eliminar']

const inicializarModulos = () => {
  return Object.keys(modulosMap).map((nombre) => ({
    nombre,
    permisosSeleccionados: [],
  }))
}

const role = ref({
  nombre: '',
  descripcion: '',
  modulos: inicializarModulos(),
})

const cancelar = () => {
  // Redirigir a la vista de roles
  router.push('/roles')
}

const guardar = async () => {
  // Imprimimos en consola el rol a guardar
  console.log('Rol a guardar:', role.value)

  // Crear el objeto en el formato requerido
  const roleData = {
    Nombre: role.value.nombre,
    Descripcion: role.value.descripcion,
    Privilegios: role.value.modulos
      .map((modulo) => {
        return {
          IdPrivilegio: modulosMap[modulo.nombre] || null,
          Permisos: modulo.permisosSeleccionados.map((permisoSeleccionado) => {
            return {
              IdPermiso: permisos.indexOf(permisoSeleccionado) + 1, // Mapeo del índice de permiso + 1 como IdPrivilegio
            }
          }),
        }
      })
      .filter((privilegio) => privilegio.IdPrivilegio !== null && privilegio.Permisos.length > 0), // Filtra los permisos sin IdPermiso o sin privilegios
  }

  // Imprimimos en consola los datos del rol formateados
  console.log('Datos del rol formateados:', roleData)

  // Imprimimos en consola los datos del rol formateados
  console.log('Datos del rol formateados:', roleData)
  await addRol(roleData)
  // Redirigir a la vista de roles
  router.push('/roles')
}
</script>

<style scoped>
.table-container {
  width: 100%;
  overflow-x: auto;
  margin-top: 1rem;
}

.table-auto {
  width: 100%;
  border-collapse: collapse;
}

th,
td {
  border: 1px solid #e5e7eb;
}
</style>
