<script setup>
import { computed, ref, watch } from 'vue'

const props = defineProps({
  selectedRole: {
    type: String,
    default: 'produtor'
  },
  loginError: {
    type: String,
    default: ''
  }
})

const emit = defineEmits(['login'])

const email = ref('')
const password = ref('')
const userType = ref(props.selectedRole || 'produtor')
const localError = ref('')
const loading = ref(false)

watch(
  () => props.selectedRole,
  (value) => {
    if (value) {
      userType.value = value
    }
  },
  { immediate: true }
)

const error = computed(() => localError.value || props.loginError || '')

const handleLogin = async () => {
  localError.value = ''
  if (!email.value || !password.value) {
    localError.value = 'Preencha o e-mail e a senha.'
    return
  }

  loading.value = true

  await new Promise((resolve) => setTimeout(resolve, 500))

  emit('login', {
    email: email.value,
    password: password.value,
    type: userType.value
  })

  loading.value = false
}

const handleForgotPassword = () => {
  alert('Fluxo de recuperação de senha')
}
</script>

<template>
  <main class="login-page">

    <!-- ================================= -->
    <!-- LOGO -->
    <!-- ================================= -->

    <section class="branding">

      <div class="branding-content">

        <img
          src="/agrovision-logo.png"
          alt="AgroVision"
          class="logo"
        />

      </div>

    </section>


    <!-- ================================= -->
    <!-- ÁREA DO LOGIN -->
    <!-- ================================= -->

    <section class="login-section">

      <form
        class="login-card"
        @submit.prevent="handleLogin"
      >

        <!-- ================================= -->
        <!-- TIPO DE USUÁRIO -->
        <!-- ================================= -->

        <div class="user-type">

          <button
            type="button"
            :class="{ active: userType === 'produtor' }"
            @click="userType = 'produtor'"
          >
            Produtor
          </button>

          <button
            type="button"
            :class="{ active: userType === 'admin' }"
            @click="userType = 'admin'"
          >
            Administrador
          </button>

        </div>


        <!-- ================================= -->
        <!-- EMAIL -->
        <!-- ================================= -->

        <div class="field">

          <label for="email">
            Email
          </label>

          <input
            id="email"
            v-model="email"
            type="email"
            autocomplete="email"
          />

        </div>


        <!-- ================================= -->
        <!-- SENHA -->
        <!-- ================================= -->

        <div class="field">

          <label for="password">
            Senha
          </label>

          <input
            id="password"
            v-model="password"
            type="password"
            autocomplete="current-password"
          />

        </div>


        <!-- ================================= -->
        <!-- ERRO -->
        <!-- ================================= -->

        <p
          v-if="error"
          class="error"
        >
          {{ error }}
        </p>


        <!-- ================================= -->
        <!-- BOTÃO ENTRAR -->
        <!-- ================================= -->

        <button
          type="submit"
          class="login-button"
          :disabled="loading"
        >
          {{ loading ? 'Entrando...' : 'Entrar' }}
        </button>


        <!-- ================================= -->
        <!-- ESQUECEU A SENHA -->
        <!-- ================================= -->

        <button
          type="button"
          class="forgot-password"
          @click="handleForgotPassword"
        >
          Esqueceu a senha?
        </button>

      </form>

    </section>

  </main>
</template>


<style scoped>

/* ================================= */
/* RESET */
/* ================================= */

* {
  box-sizing: border-box;
}


/* ================================= */
/* PÁGINA */
/* ================================= */

.login-page {
  min-height: 100vh;

  display: grid;

  grid-template-columns: 1fr 1fr;

  background: #ffffff;
}


/* ================================= */
/* LADO ESQUERDO */
/* ================================= */

.branding {
  display: flex;

  align-items: center;
  justify-content: center;

  background: #ffffff;
}


.branding-content {
  display: flex;

  flex-direction: column;

  align-items: center;
  justify-content: center;
}


/* ================================= */
/* LOGO */
/* ================================= */

.logo {
  width: 290px;

  height: auto;

  object-fit: contain;
}


/* ================================= */
/* LADO DIREITO */
/* ================================= */

.login-section {
  display: flex;

  align-items: center;
  justify-content: center;

  background: #6db45c;
}


/* ================================= */
/* CARD DE LOGIN */
/* ================================= */

.login-card {
  width: 308px;

  padding: 15px 14px 13px;

  background: #ffffff;

  border-radius: 5px;

  box-sizing: border-box;
}


/* ================================= */
/* TIPO DE USUÁRIO */
/* ================================= */

.user-type {
  display: grid;

  grid-template-columns: 1fr 1fr;

  gap: 5px;

  padding: 4px;

  margin-bottom: 18px;

  background: #f1f1f1;

  border-radius: 5px;
}


.user-type button {
  height: 28px;

  border: none;

  border-radius: 4px;

  background: transparent;

  color: #555555;

  font-size: 10px;

  cursor: pointer;

  transition:
    background 0.2s,
    color 0.2s;
}


.user-type button:hover {
  color: #222222;
}


.user-type button.active {
  background: #6db45c;

  color: #ffffff;

  font-weight: 600;
}


/* ================================= */
/* CAMPOS */
/* ================================= */

.field {
  display: flex;

  flex-direction: column;

  margin-bottom: 14px;
}


.field label {
  margin-bottom: 5px;

  font-family: Arial, Helvetica, sans-serif;

  font-size: 10px;

  color: #222222;
}


.field input {
  width: 100%;

  height: 25px;

  padding: 0 8px;

  border: 1px solid #dddddd;

  border-radius: 5px;

  outline: none;

  font-family: Arial, Helvetica, sans-serif;

  font-size: 12px;

  color: #222222;

  background: #ffffff;

  transition: border-color 0.2s;
}


.field input:focus {
  border-color: #6db45c;
}


/* ================================= */
/* ERRO */
/* ================================= */

.error {
  margin: -4px 0 8px;

  color: #c62828;

  font-family: Arial, Helvetica, sans-serif;

  font-size: 10px;

  line-height: 14px;
}


/* ================================= */
/* BOTÃO ENTRAR */
/* ================================= */

.login-button {
  width: 100%;

  height: 24px;

  border: none;

  border-radius: 4px;

  background: #292929;

  color: #ffffff;

  font-family: Arial, Helvetica, sans-serif;

  font-size: 10px;

  cursor: pointer;

  transition:
    background 0.2s,
    transform 0.1s;
}


.login-button:hover {
  background: #1c1c1c;
}


.login-button:active {
  transform: scale(0.99);
}


.login-button:disabled {
  opacity: 0.7;

  cursor: not-allowed;
}


/* ================================= */
/* ESQUECEU A SENHA */
/* ================================= */

.forgot-password {
  display: block;

  margin: 14px auto 0;

  padding: 0;

  border: none;

  background: transparent;

  color: #333333;

  font-family: Arial, Helvetica, sans-serif;

  font-size: 10px;

  text-decoration: underline;

  cursor: pointer;
}


.forgot-password:hover {
  color: #000000;
}


/* ================================= */
/* TABLETS / CELULARES */
/* ================================= */

@media (max-width: 768px) {

  .login-page {
    min-height: 100svh;

    display: flex;

    flex-direction: column;

    background: #6db45c;
  }


  /* =============================== */
  /* ÁREA DA LOGO */
  /* =============================== */

  .branding {
    min-height: auto;

    padding: 42px 24px 28px;

    background: #ffffff;

    border-radius: 0 0 28px 28px;
  }


  .branding-content {
    width: 100%;
  }


  .logo {
    width: 190px;

    max-width: 100%;
  }


  /* =============================== */
  /* ÁREA DO LOGIN */
  /* =============================== */

  .login-section {
    flex: 1;

    min-height: 0;

    width: 100%;

    /*
     * Espaço entre a logo e o card.
     * Aumente ou diminua este valor
     * conforme preferir.
     */
    padding: 90px 20px 36px;

    background: #6db45c;

    align-items: flex-start;

    justify-content: center;
  }


  /* =============================== */
  /* CARD */
  /* =============================== */

  .login-card {
    width: 100%;

    max-width: 380px;

    margin: 0 auto;

    padding: 24px 20px 22px;

    background: #ffffff;

    border-radius: 16px;

    box-shadow: 0 8px 30px rgba(0, 0, 0, 0.08);
  }


  /* =============================== */
  /* TIPO DE USUÁRIO */
  /* =============================== */

  .user-type {
    gap: 4px;

    padding: 4px;

    margin-bottom: 24px;

    border-radius: 8px;
  }


  .user-type button {
    height: 38px;

    border-radius: 6px;

    font-size: 12px;
  }


  /* =============================== */
  /* CAMPOS */
  /* =============================== */

  .field {
    margin-bottom: 18px;
  }


  .field label {
    margin-bottom: 7px;

    font-size: 12px;
  }


  .field input {
    height: 44px;

    padding: 0 12px;

    border-radius: 8px;

    font-size: 14px;
  }


  /* =============================== */
  /* ERRO */
  /* =============================== */

  .error {
    margin: -4px 0 12px;

    font-size: 11px;

    line-height: 16px;
  }


  /* =============================== */
  /* BOTÃO */
  /* =============================== */

  .login-button {
    height: 44px;

    border-radius: 8px;

    font-size: 13px;
  }


  /* =============================== */
  /* ESQUECEU A SENHA */
  /* =============================== */

  .forgot-password {
    margin: 16px auto 0;

    font-size: 11px;
  }

}


/* ================================= */
/* CELULARES PEQUENOS */
/* ================================= */

@media (max-width: 400px) {

  /* =============================== */
  /* LOGO */
  /* =============================== */

  .branding {
    padding: 30px 20px 22px;

    border-radius: 0 0 22px 22px;
  }


  .logo {
    width: 155px;
  }


  /* =============================== */
  /* ÁREA DO LOGIN */
  /* =============================== */

  .login-section {
    /*
     * Mantém um bom espaço mesmo
     * em celulares menores.
     */
    padding: 45px 16px 28px;
  }


  /* =============================== */
  /* CARD */
  /* =============================== */

  .login-card {
    padding: 20px 16px 18px;

    border-radius: 14px;
  }


  /* =============================== */
  /* TIPO DE USUÁRIO */
  /* =============================== */

  .user-type {
    margin-bottom: 20px;
  }


  .user-type button {
    height: 36px;

    font-size: 11px;
  }


  /* =============================== */
  /* CAMPOS */
  /* =============================== */

  .field {
    margin-bottom: 15px;
  }


  .field input {
    height: 42px;

    font-size: 14px;
  }


  /* =============================== */
  /* BOTÃO */
  /* =============================== */

  .login-button {
    height: 42px;

    font-size: 12px;
  }


  /* =============================== */
  /* ESQUECEU A SENHA */
  /* =============================== */

  .forgot-password {
    font-size: 10px;
  }

}

</style>