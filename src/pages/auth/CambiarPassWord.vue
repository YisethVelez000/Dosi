<template>
  <VaForm ref="passwordForm" @submit.prevent="submit">
    <h1 class="font-semibold text-4xl mb-4">Cambiar contraseña</h1>
    <p class="text-base mb-4 leading-5">Cambie su contraseña</p>
    <VaInput
      v-model="contraseña"
      :rules="[(v: any) => !!v || 'La contraseña es requerida']"
      class="mb-4"
      label="Contraseña"
      type="password"
    />
    <VaInput
      v-model="ConfirmarContraseña"
      :rules="[
        (v: any) => !!v || 'Confirmar contraseña es requerida',
        (v: string) => v === contraseña.valueOf() || 'Las contraseñas no coinciden',
      ]"
      class="mb-4"
      label="Confirmar contraseña"
      type="password"
    />
    <VaButton class="w-full mb-2" type="submit" preset="primary" @click="submit">Cambiar contraseña</VaButton>
    <VaButton :to="{ name: 'login' }" class="w-full" preset="secondary">Cancelar</VaButton>
  </VaForm>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { useRouter } from 'vue-router'

// Definir las props que el componente recibirá
const props = defineProps<{
  IdUsuario: string
  token: string
}>()
const IdUsuario = props.IdUsuario
const token = props.token

const contraseña = ref('')
const ConfirmarContraseña = ref('')
const router = useRouter()

const submit = () => {
  console.log(props.IdUsuario)
  console.log(props.token)
  console.log('Enviando...')
  // Send email to recover password
  fetch(`https://apinodedvp.onrender.com/usuarios/recuperaContrasena/${token}/${IdUsuario}`, {
    method: 'PUT',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({ Contrasena: contraseña.value }),
  })
    .then((response) => {
      if (response.ok) {
        router.push({ name: 'login' })
      }
    })
    .catch((error) => {
      console.log(error)
    })
}
</script>
