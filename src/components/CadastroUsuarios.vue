<script setup>
import { computed, ref, reactive } from 'vue'
import { authMock } from '../mocks/auth'
import CadastroLavoura from './CadastroLavoura.vue'

const props = defineProps({
  userName: {
    type: String,
    default: 'Usuário'
  },
  userType: {
    type: String,
    default: 'admin'
  }
})

const emit = defineEmits(['logout'])

// sidebar state for mobile (open/closed)
const isSidebarOpen = ref(false)
const toggleSidebar = () => { isSidebarOpen.value = !isSidebarOpen.value }

const adminMenuItems = [
  { label: 'Cadastrar Usuários', active: true, icon: '👤' },
  { label: 'Defensivos', active: false, icon: '🛡️' },
  { label: 'Cadastrar Lavouras', active: false, icon: '🌾' },
  { label: 'Cadastrar Orientação', active: false, icon: '📘' }
]

const producerMenuItems = [
  { label: 'Minhas Lavouras', active: true, icon: '🌿' },
  { label: 'Orientação', active: false, icon: '📘' }
]

const menuItems = computed(() => (props.userType === 'produtor' ? producerMenuItems : adminMenuItems))

const activeView = ref('Cadastrar Usuários')
const setActive = (label) => { activeView.value = label }

const producers = computed(() => users.value.filter(u => u.type === 'produtor'))

// Users management (local state initialized from mock)
const users = ref([])
// initialize a shallow copy so we don't mutate the mock directly
users.value = authMock.map((u) => ({ ...u }))

// (show all users in the list; profile is chosen per-user in the form)

const nextId = () => Math.max(0, ...users.value.map((u) => u.id || 0)) + 1

const form = reactive({ id: null, name: '', email: '', password: '', type: '' })
const editingId = ref(null)

const resetForm = () => {
  form.id = null
  form.name = ''
  form.email = ''
  form.password = ''
  form.type = ''
  editingId.value = null
}

const startEdit = (u) => {
  editingId.value = u.id
  form.id = u.id
  form.name = u.name
  form.email = u.email
  form.password = u.password || ''
  form.type = u.type || 'produtor'
}

const submitUser = () => {
  if (!form.name || !form.email || !form.password) {
    alert('Preencha nome, email e senha.')
    return
  }

  if (editingId.value === null) {
    // create
    users.value.push({ id: nextId(), name: form.name, email: form.email, password: form.password, type: form.type || 'produtor' })
    resetForm()
    return
  }

  // update
  const idx = users.value.findIndex((x) => x.id === editingId.value)
  if (idx !== -1) {
    users.value[idx].name = form.name
    users.value[idx].email = form.email
    users.value[idx].password = form.password
    users.value[idx].type = form.type || users.value[idx].type
  }

  resetForm()
}

const deleteUser = (id) => {
  if (!confirm('Tem certeza que deseja excluir este usuário?')) return
  users.value = users.value.filter((u) => u.id !== id)
}
</script>

<template>
  <div class="app-shell">
    <aside :class="['sidebar', { 'mobile-closed': !isSidebarOpen }]">
      <div class="user-status">
        <div class="user-avatar" aria-label="Foto do usuário">
          <span>👤</span>
        </div>

        <div class="user-meta">
          <span class="status-label">Conectado</span>
          <span class="status-user">{{ userName }}</span>

          <!-- profile selection moved to the cadastro form -->
        </div>
      </div>

      <nav class="menu" aria-label="Menu principal">
        <button
          v-for="item in menuItems"
          :key="item.label"
          type="button"
          class="menu-item"
          :class="{ active: activeView === item.label }"
          @click="setActive(item.label)"
        >
          <span class="item-icon" aria-hidden="true">{{ item.icon }}</span>
          <span>{{ item.label }}</span>
        </button>
      </nav>

      <div class="menu-section-divider"></div>
      <button type="button" class="menu-item secondary" @click="emit('logout')">
        <span class="item-icon" aria-hidden="true">🚪</span>
        <span>Sair</span>
      </button>
    </aside>

    <!-- backdrop for mobile when sidebar is open -->
    <div v-if="isSidebarOpen" class="backdrop" @click="toggleSidebar"></div>

    <main class="content-area">
      <!-- hamburger toggle placed inside content so title appears below it -->
      <button class="hamburger" :class="{ hidden: isSidebarOpen }" @click="toggleSidebar" aria-label="Abrir menu">
        <svg width="20" height="20" viewBox="0 0 24 24" fill="none" aria-hidden>
          <path d="M3 6h18M3 12h18M3 18h18" stroke="#000" stroke-width="1.6" stroke-linecap="round"/>
        </svg>
      </button>

      <h1 class="page-title">{{ activeView === 'Cadastrar Lavouras' ? 'Lavouras' : (props.userType === 'produtor' ? 'Minhas Lavouras' : 'Usuários') }}</h1>

      <div v-if="activeView === 'Cadastrar Lavouras'">
        <CadastroLavoura :producers="producers" />
      </div>

      <div v-else-if="props.userType === 'admin'" class="admin-users">
        <section class="user-form">
          <h2 class="form-title">{{ editingId === null ? 'Cadastrar usuário' : 'Editar usuário' }}</h2>

          <form @submit.prevent="submitUser" class="nice-form">
            <div class="field input-with-icon">
              <svg class="input-icon" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg" aria-hidden>
                <path d="M12 12c2.761 0 5-2.239 5-5s-2.239-5-5-5-5 2.239-5 5 2.239 5 5 5z" stroke="#000" stroke-width="1.2" stroke-linecap="round" stroke-linejoin="round"/>
                <path d="M20 21v-1c0-2.761-4.03-5-8-5s-8 2.239-8 5v1" stroke="#000" stroke-width="1.2" stroke-linecap="round" stroke-linejoin="round"/>
              </svg>
              <input v-model="form.name" type="text" placeholder="Nome completo" />
            </div>

            <div class="field input-with-icon">
              <svg class="input-icon" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg" aria-hidden>
                <path d="M3 8l9 6 9-6" stroke="#000" stroke-width="1.2" stroke-linecap="round" stroke-linejoin="round"/>
                <path d="M21 8v8a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V8" stroke="#000" stroke-width="1.2" stroke-linecap="round" stroke-linejoin="round"/>
              </svg>
              <input v-model="form.email" type="email" placeholder="email@exemplo.com" />
            </div>


            <div class="field input-with-icon">
              <svg class="input-icon" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg" aria-hidden>
                <rect x="3" y="11" width="18" height="11" rx="2" stroke="#000" stroke-width="1.2" stroke-linecap="round" stroke-linejoin="round"/>
                <path d="M7 11V8a5 5 0 0 1 10 0v3" stroke="#000" stroke-width="1.2" stroke-linecap="round" stroke-linejoin="round"/>
              </svg>
              <input v-model="form.password" type="password" placeholder="Senha" />
            </div>

              <div class="field">
                <select id="user-profile" v-model="form.type" class="profile-select" aria-label="Perfil do usuário">
                  <option disabled value="">Perfil</option>
                  <option value="produtor">Produtor</option>
                  <option value="admin">Administrador</option>
                </select>
              </div>
            <div class="form-actions">
              <button class="primary-btn" type="submit">
                <svg width="16" height="16" viewBox="0 0 24 24" fill="none" aria-hidden>
                  <path d="M12 5v14M5 12h14" stroke="#fff" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/>
                </svg>
                <span>{{ editingId === null ? 'Cadastrar' : 'Salvar' }}</span>
              </button>

              <button type="button" class="ghost-btn" v-if="editingId !== null" @click="resetForm">Cancelar</button>
            </div>
          </form>
        </section>

        <section class="user-list">
          <h2>Lista de usuários</h2>

          <table class="users-table">
            <thead>
              <tr>
                <th>Nome</th>
                <th>Email</th>
                <th>Perfil</th>
                <th></th>
              </tr>
            </thead>

            <tbody>
              <tr v-for="u in users" :key="u.id">
                <td>{{ u.name }}</td>
                <td>{{ u.email }}</td>
                <td>{{ u.type }}</td>
                <td style="white-space:nowrap">
                  <button class="icon-btn" title="Editar" aria-label="Editar" @click="startEdit(u)">
                    <svg width="16" height="16" viewBox="0 0 24 24" fill="none" aria-hidden>
                      <path d="M3 21l3-1 11-11 1-3-3 1L4 20z" stroke="#000" stroke-width="1.2" stroke-linecap="round" stroke-linejoin="round"/>
                    </svg>
                  </button>

                  <button class="icon-btn danger" title="Excluir" aria-label="Excluir" @click="deleteUser(u.id)">
                    <svg width="16" height="16" viewBox="0 0 24 24" fill="none" aria-hidden>
                      <path d="M3 6h18" stroke="#000" stroke-width="1.2" stroke-linecap="round" stroke-linejoin="round"/>
                      <path d="M8 6v14a2 2 0 0 0 2 2h4a2 2 0 0 0 2-2V6" stroke="#000" stroke-width="1.2" stroke-linecap="round" stroke-linejoin="round"/>
                      <path d="M10 11v6M14 11v6" stroke="#000" stroke-width="1.2" stroke-linecap="round" stroke-linejoin="round"/>
                    </svg>
                  </button>
                </td>
              </tr>
            </tbody>
          </table>
        </section>
      </div>

      <div v-else>
        <p>Área do produtor: aqui aparecem suas lavouras.</p>
      </div>
    </main>
  </div>
</template>

<style scoped>
/* copied and adjusted styles from previous file, keep consistent */
:global(html, body, #app) {
  margin: 0;
  height: 100vh;
  overflow: hidden; /* prevent page-level scrolling */
}

:global(body) {
  font-family: Arial, Helvetica, sans-serif;
  background: #efefef;
  margin: 0;
}

* {
  box-sizing: border-box;
}

.app-shell {
  display: flex;
  width: 100%;
  height: 100vh;
  overflow: hidden; /* no page scroll; internal areas handle overflow */
  background: #efefef;
  position: relative;
}

.sidebar {
  width: 270px;
  background: linear-gradient(180deg, #66bc66 0%, #4dae52 100%);
  color: #111111;
  padding-top: 14px;
  border-right: 1px solid rgba(0, 0, 0, 0.08);
  box-shadow: inset -1px 0 0 rgba(255, 255, 255, 0.15);
}

.user-status {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 8px 18px 16px;
  color: rgba(17, 17, 17, 0.9);
}

.profile-select {
  padding: 6px 8px;
  border-radius: 8px;
  border: 1px solid #e6e6e6;
  background: #fff;
  font-size: 13px;
}

.user-avatar {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background: #f1f1f1;
  border: 2px solid rgba(17, 17, 17, 0.25);
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 22px;
  color: #000000;
  flex-shrink: 0;
}

.user-meta {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.status-label { font-size: 14px; font-weight: 600 }
.status-user { font-size: 15px; font-weight: 700; letter-spacing: 0.2px }

.menu { display:flex; flex-direction:column }
.menu-item { display:flex; align-items:center; gap:12px; width:100%; border:none; background:transparent; color:#111; text-align:left; padding:15px 18px 15px 22px; font-size:17px; font-weight:500; cursor:pointer }
.menu-item:hover { background: rgba(255,255,255,0.08) }
.item-icon { width:18px; display:inline-flex; justify-content:center; align-items:center; font-size:18px; color:#000; }

.menu-section-divider { height:1px; background: rgba(17,17,17,0.22); margin:12px 18px 8px }

.backdrop { position: fixed; inset:0; background: rgba(0,0,0,0.36); z-index:80; display:none }
.hidden { display:none !important }
.hamburger { display:none; position:fixed; left:12px; top:12px; z-index:110; width:44px; height:44px; border-radius:10px; border:none; background:#fff; align-items:center; justify-content:center; box-shadow: 0 6px 18px rgba(0,0,0,0.08); cursor:pointer }

@media (max-width: 900px) {
  .sidebar { position:fixed; left:0; top:0; bottom:0; transform:translateX(-100%); transition:transform 220ms ease; z-index:100; width:260px; max-width:85% }
  .sidebar.mobile-closed { transform: translateX(-100%) }
  .sidebar:not(.mobile-closed) { transform: translateX(0); box-shadow: 6px 0 18px rgba(0,0,0,0.18) }
  .backdrop { display:block }
  .hamburger { display:inline-flex; top: 18px }
  .content-area { padding: 72px 18px 28px 76px }
  .admin-users { grid-template-columns: 1fr }
  .admin-users .user-form, .admin-users .user-list { width:100% }
}

@media (max-width: 700px) {
  .admin-users { gap:14px }
}

@media (max-width: 1000px) {
  .admin-users { grid-template-columns: 1fr !important; display: grid; grid-auto-flow: row }
  .admin-users .user-form { order: 1 }
  .admin-users .user-list { order: 2; margin-top: 12px }
  .admin-users .user-form, .admin-users .user-list { width:100%; display:block }
  /* ensure hamburger does not overlap title on intermediate sizes */
  .content-area { padding-left: 76px }
}

.content-area { flex:1; background:#f4f4f4; padding:28px 32px; overflow:auto; max-height:100vh }
.content-area h1 { margin:0; color:#2d2d2d; font-size:32px; font-weight:700; letter-spacing:-0.4px }

.admin-users { display:grid; grid-template-columns:360px 1fr; gap:24px; margin-top:20px }
.user-form { background:#fff; padding:16px; border-radius:8px; box-shadow:0 6px 18px rgba(0,0,0,0.04) }
.user-list { background:#fff; padding:16px; border-radius:8px; box-shadow:0 6px 18px rgba(0,0,0,0.04) }
.users-table { width:100%; border-collapse:collapse }
.users-table th, .users-table td { text-align:left; padding:8px 10px; border-bottom:1px solid #eee }
.users-table thead th { font-weight:600; font-size:13px }
.users-table tbody tr:last-child td { border-bottom:none }
.users-table button { margin-left:6px; padding:6px 8px; background:transparent; border:1px solid #ddd; border-radius:6px; cursor:pointer }
.input-with-icon { display:flex; align-items:center; gap:10px; background:#fff; padding:12px 14px; border-radius:10px; border:1px solid #e9e9e9; box-shadow:0 10px 24px rgba(8,12,8,0.04) }
.input-with-icon .input-icon { width:18px; height:18px; flex:0 0 18px; opacity:0.9 }
.input-with-icon input { border:none; outline:none; font-size:15px; background:transparent; width:100% }
.input-with-icon input::placeholder { color:#cfcfcf }
.form-title { margin:0 0 12px 0; font-size:18px; color:#333 }
.form-actions { display:flex; gap:10px; align-items:center; margin-top:12px }
.primary-btn { display:inline-flex; gap:8px; align-items:center; padding:10px 14px; background:#2b8a3e; color:#fff; border:none; border-radius:10px; cursor:pointer; box-shadow:0 8px 20px rgba(43,138,62,0.08) }
.ghost-btn { background:transparent; border:1px solid #e6e6e6; color:#333; padding:8px 12px; border-radius:8px }
.icon-btn { display:inline-flex; align-items:center; justify-content:center; min-width:44px; height:36px; border-radius:8px; background:#fff; border:1px solid #e5e5e5; margin-left:6px; cursor:pointer }
.icon-btn:hover { box-shadow:0 8px 20px rgba(0,0,0,0.04) }
.icon-btn.danger { background:#ffecec; border-color:#ffd6d6 }

/* form container styling for nicer look */
.nice-form { display:flex; flex-direction:column; gap:12px }
.nice-form .field { display:flex; flex-direction:column; gap:8px }
.nice-form select, .nice-form input { padding:10px 12px; border-radius:8px; border:1px solid #eaeaea; font-size:14px }
.nice-form input:focus, .nice-form select:focus { outline:none; border-color:#2b8a3e; box-shadow:0 8px 20px rgba(43,138,62,0.06) }
</style>