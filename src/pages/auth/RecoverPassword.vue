<template>
  <VaForm ref="passwordForm" @submit.prevent="submit">
    <h1 class="font-semibold text-4xl mb-4">¿Olvido su contraseña?</h1>
    <p class="text-base mb-4 leading-5">
      Si olvido su contraseña, ingrese su correo electrónico y le enviaremos un enlace para restablecer su contraseña.
    </p>
    <VaInput
      v-model="email"
      :rules="[
        (v: any) => !!v || 'El correo es requerido',
        (v: any) => /.+@.+\..+/.test(v) || 'El correo debe ser válido',
        async (v: any) => (await validateEmail()) || 'El correo no existe',
      ]"
      class="mb-4"
      label="Correo electrónico"
      type="email"
    />
    <VaButton class="w-full mb-2" @click="submit">Enviar </VaButton>
    <VaButton :to="{ name: 'login' }" class="w-full" preset="secondary" @click="submit">Cancelar</VaButton>
  </VaForm>
</template>

<script lang="ts" setup>
import { ref } from 'vue'
import { useForm } from 'vuestic-ui'
import { useRouter } from 'vue-router'

const email = ref('')
const form = useForm('passwordForm')
const router = useRouter()

const submit = () => {
  if (form.validate()) {
    // Send email to recover password
    const response = fetch('https://apinodedvp.onrender.com/usuarios/recuperaContrasena', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({ CorreoElectronico: email.value }),
    })
    response.then((res) => {
      if (res.ok) {
        router.push({ name: 'login' })
      }
    })
  }
}

// Función para conocer si el correo electrónico ya existe
const validateEmail = async () => {
  const response = await fetch('https://apinodedvp.onrender.com/usuarios/validateEmail', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      CorreoElectronico: email.value,
    }),
  })
  if (response.status === 200) {
    return false
  } else {
    return true
  }
}
</script>
