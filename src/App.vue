<script setup>
import { computed, ref } from 'vue'
import Login from './components/Login.vue'
import CadastroUsuarios from './components/CadastroUsuarios.vue'
import { authMock } from './mocks/auth'

const currentUser = ref(null)
const selectedRole = ref('produtor')
const loginError = ref('')

const isLoggedIn = computed(() => !!currentUser.value)

const handleLogin = (user) => {
  currentUser.value = user
  selectedRole.value = user.type
  loginError.value = ''
}

const handleLogout = () => {
  currentUser.value = null
  loginError.value = ''
}

const handleLoginAttempt = ({ email, password, type }) => {
  const user = authMock.find(
    (item) => item.email === email && item.password === password && item.type === type
  )

  if (!user) {
    loginError.value = 'E-mail, senha ou perfil inválido.'
    return
  }

  handleLogin(user)
}
</script>

<template>
  <Login
    v-if="!isLoggedIn"
    :selected-role="selectedRole"
    :login-error="loginError"
    @login="handleLoginAttempt"
  />

  <CadastroUsuarios
    v-else
    :user-name="currentUser?.name || 'Usuário'"
    :user-type="currentUser?.type || 'admin'"
    @logout="handleLogout"
  />
</template>
