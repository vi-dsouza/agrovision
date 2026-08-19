<script setup>
import { computed, reactive, ref, watch } from 'vue'
import { authMock } from '../mocks/auth'

const props = defineProps({
  producerId: {
    type: [Number, String, null],
    default: null
  },
  isProducerView: {
    type: Boolean,
    default: false
  }
})

const STORAGE_KEY = 'agrovision:orientacoes'
const USERS_STORAGE_KEY = 'agrovision:users'
const LAVOURAS_STORAGE_KEY = 'agrovision:lavouras'
const DEFENSIVOS_STORAGE_KEY = 'agrovision:defensivos'

const readStored = (key, fallback = []) => {
  try {
    const saved = localStorage.getItem(key)
    return saved ? JSON.parse(saved) : fallback
  } catch {
    return fallback
  }
}

const users = ref(readStored(USERS_STORAGE_KEY, authMock.map((user) => ({ ...user }))))
const lavouras = ref(readStored(LAVOURAS_STORAGE_KEY, []))
const defensivos = ref(readStored(DEFENSIVOS_STORAGE_KEY, []))
const orientacoes = ref(readStored(STORAGE_KEY, []))

watch(users, (value) => {
  try { localStorage.setItem(USERS_STORAGE_KEY, JSON.stringify(value)) } catch {}
}, { deep: true })

watch(lavouras, (value) => {
  try { localStorage.setItem(LAVOURAS_STORAGE_KEY, JSON.stringify(value)) } catch {}
}, { deep: true })

watch(defensivos, (value) => {
  try { localStorage.setItem(DEFENSIVOS_STORAGE_KEY, JSON.stringify(value)) } catch {}
}, { deep: true })

watch(orientacoes, (value) => {
  try { localStorage.setItem(STORAGE_KEY, JSON.stringify(value)) } catch {}
}, { deep: true })

const producers = computed(() => users.value.filter((user) => user.type === 'produtor'))
const getProducerById = (id) => users.value.find((user) => Number(user.id) === Number(id)) || null
const getProducerLavouras = (producerId) => lavouras.value.filter((l) => Number(l.produtorId) === Number(producerId))
const getDefensivoById = (id) => defensivos.value.find((item) => Number(item.id) === Number(id)) || null

const form = reactive({
  id: null,
  producerId: props.producerId ?? '',
  defensivoId: '',
  dose: '',
  recomendacao: '',
  proximaVisita: '',
  observacoes: ''
})

const editingId = ref(null)

const resetForm = () => {
  form.id = null
  form.producerId = props.producerId ?? ''
  form.defensivoId = ''
  form.dose = ''
  form.recomendacao = ''
  form.proximaVisita = ''
  form.observacoes = ''
  editingId.value = null
}

const nextId = () => Math.max(0, ...orientacoes.value.map((item) => item.id || 0)) + 1

const startEdit = (item) => {
  editingId.value = item.id
  form.id = item.id
  form.producerId = item.producerId
  form.defensivoId = item.defensivoId
  form.dose = item.dose
  form.recomendacao = item.recomendacao
  form.proximaVisita = item.proximaVisita
  form.observacoes = item.observacoes || ''
}

const submitOrientacao = () => {
  if (!form.producerId || !form.defensivoId || !form.dose || !form.recomendacao || !form.proximaVisita) {
    alert('Preencha produtor, defensivo, dose, recomendação e próxima visita técnica.')
    return
  }

  const payload = {
    id: form.id ?? nextId(),
    producerId: Number(form.producerId),
    defensivoId: Number(form.defensivoId),
    dose: form.dose,
    recomendacao: form.recomendacao,
    proximaVisita: form.proximaVisita,
    observacoes: form.observacoes
  }

  if (editingId.value === null) {
    orientacoes.value.push(payload)
  } else {
    const idx = orientacoes.value.findIndex((item) => item.id === editingId.value)
    if (idx !== -1) {
      orientacoes.value[idx] = payload
    }
  }

  resetForm()
}

const deleteOrientacao = (id) => {
  if (!confirm('Deseja excluir esta orientação?')) return
  orientacoes.value = orientacoes.value.filter((item) => item.id !== id)
}

const producerList = computed(() => {
  if (props.isProducerView && props.producerId) {
    return producers.value.filter((producer) => Number(producer.id) === Number(props.producerId))
  }

  return producers.value
})

const visibleOrientacoes = computed(() => {
  if (props.isProducerView && props.producerId) {
    return orientacoes.value.filter((item) => Number(item.producerId) === Number(props.producerId))
  }

  return orientacoes.value
})

const parseDateValue = (value) => {
  if (!value) return null
  const candidate = new Date(`${value}T12:00:00`)
  return Number.isNaN(candidate.getTime()) ? new Date(value) : candidate
}

const todayStart = () => {
  const now = new Date()
  now.setHours(0, 0, 0, 0)
  return now
}

const upcomingOrientacoes = computed(() =>
  visibleOrientacoes.value.filter((item) => {
    if (!item.proximaVisita) return false
    const date = parseDateValue(item.proximaVisita)
    if (!date || Number.isNaN(date.getTime())) return false
    date.setHours(0, 0, 0, 0)
    return date.getTime() >= todayStart().getTime()
  })
)

const sortedOrientacoes = computed(() =>
  [...upcomingOrientacoes.value].sort((a, b) => {
    const da = a.proximaVisita ? parseDateValue(a.proximaVisita).getTime() : Number.MAX_SAFE_INTEGER
    const db = b.proximaVisita ? parseDateValue(b.proximaVisita).getTime() : Number.MAX_SAFE_INTEGER
    return da - db
  })
)

const nextVisit = computed(() => sortedOrientacoes.value[0] || null)

const nextVisitDays = computed(() => {
  if (!nextVisit.value?.proximaVisita) return null

  const today = new Date()
  today.setHours(0, 0, 0, 0)

  const target = parseDateValue(nextVisit.value.proximaVisita)
  target.setHours(0, 0, 0, 0)

  const diff = Math.ceil((target.getTime() - today.getTime()) / (1000 * 60 * 60 * 24))
  if (diff === 0) return 'Hoje'
  if (diff === 1) return 'Em 1 dia'
  return `Em ${diff} dias`
})

const producerCount = computed(() => new Set(visibleOrientacoes.value.map((item) => Number(item.producerId))).size)

const visitTrend = computed(() => {
  const months = Array.from({ length: 4 }, (_, index) => {
    const date = new Date()
    date.setMonth(date.getMonth() - (3 - index))
    return {
      label: date.toLocaleDateString('pt-BR', { month: 'short' }).replace('.', ''),
      month: date.getMonth(),
      year: date.getFullYear()
    }
  })

  const maxValue = Math.max(
    1,
    ...months.map(({ month, year }) => visibleOrientacoes.value.filter((item) => {
      if (!item.proximaVisita) return false
      const date = parseDateValue(item.proximaVisita)
      return date.getMonth() === month && date.getFullYear() === year
    }).length)
  )

  return months.map((entry) => {
    const value = visibleOrientacoes.value.filter((item) => {
      if (!item.proximaVisita) return false
      const date = parseDateValue(item.proximaVisita)
      return date.getMonth() === entry.month && date.getFullYear() === entry.year
    }).length

    return {
      ...entry,
      value,
      height: `${Math.max(12, (value / maxValue) * 100)}%`
    }
  })
})

const formatDate = (value) => {
  if (!value) return '—'
  const date = parseDateValue(value)
  if (!date || Number.isNaN(date.getTime())) return value
  return date.toLocaleDateString('pt-BR')
}
</script>

<template>
  <div class="orientation-shell">
    <section class="orientation-dashboard">
      <div class="dashboard-card dashboard-highlight">
        <div class="dashboard-label">Próxima visita</div>
        <h3 v-if="nextVisit">{{ formatDate(nextVisit.proximaVisita) }}</h3>
        <h3 v-else>Sem visita agendada</h3>

        <div v-if="nextVisit" class="next-visit-detail">
          <span>{{ getProducerById(nextVisit.producerId)?.name || 'Produtor' }}</span>
          <span>•</span>
          <span>{{ getDefensivoById(nextVisit.defensivoId)?.nome || 'Defensivo' }}</span>
        </div>
        <div v-if="nextVisit" class="next-visit-meta">{{ nextVisitDays }}</div>
      </div>

      <div class="dashboard-card">
        <div class="dashboard-label">Total de orientações</div>
        <h3>{{ visibleOrientacoes.length }}</h3>
      </div>

      <div class="dashboard-card">
        <div class="dashboard-label">Produtores</div>
        <h3>{{ producerCount }}</h3>
      </div>

      <div class="dashboard-card">
        <div class="dashboard-label">Próximas visitas</div>
        <h3>{{ sortedOrientacoes.length }}</h3>
      </div>
    </section>

    <section class="chart-panel card">
      <div class="chart-header">
        <div>
          <h3>Visitas ao longo do tempo</h3>
          <p>Comparativo das visitas agendadas por mês.</p>
        </div>
      </div>

      <div class="chart-bars" aria-label="Gráfico de visitas">
        <div v-for="bar in visitTrend" :key="`${bar.label}-${bar.year}`" class="chart-column">
          <span class="chart-value">{{ bar.value }}</span>
          <span class="chart-bar" :style="{ height: bar.height }"></span>
          <small>{{ bar.label }}</small>
        </div>
      </div>
    </section>

    <div class="orientation-top-row">
      <section v-if="!props.isProducerView" class="orientation-form card">
        <h3>{{ editingId === null ? 'Cadastrar orientação' : 'Editar orientação' }}</h3>

        <form @submit.prevent="submitOrientacao" class="nice-form orientation-fields">
          <label class="field">
            <span>Produtor</span>
            <select v-model.number="form.producerId">
              <option value="">Selecione o produtor</option>
              <option v-for="producer in producers" :key="producer.id" :value="producer.id">{{ producer.name }}</option>
            </select>
          </label>

          <label class="field">
            <span>Defensivo recomendado</span>
            <select v-model.number="form.defensivoId">
              <option value="">Selecione o defensivo</option>
              <option v-for="defensivo in defensivos" :key="defensivo.id" :value="defensivo.id">{{ defensivo.nome }}</option>
            </select>
          </label>

          <label class="field">
            <span>Quantidade ou dose recomendada</span>
            <input v-model="form.dose" type="text" placeholder="Ex: 2 L/ha" />
          </label>

          <label class="field">
            <span>Recomendação</span>
            <textarea v-model="form.recomendacao" rows="3" placeholder="Descreva a recomendação técnica"></textarea>
          </label>

          <div class="grid-row">
            <label class="field">
              <span>Próxima visita técnica</span>
              <input v-model="form.proximaVisita" type="date" />
            </label>

            <label class="field">
              <span>Observações</span>
              <input v-model="form.observacoes" type="text" placeholder="Detalhes adicionais" />
            </label>
          </div>

          <div class="form-actions">
            <button type="submit" class="primary-btn">
              {{ editingId === null ? 'Cadastrar orientação' : 'Salvar' }}
            </button>
            <button type="button" class="ghost-btn" @click="resetForm">Limpar</button>
          </div>
        </form>
      </section>

      <section class="orientation-history card">
        <h3>{{ props.isProducerView ? 'Histórico das minhas orientações' : 'Histórico de orientações' }}</h3>

        <table class="data-table">
          <thead>
            <tr>
              <th>Produtor</th>
              <th>Defensivo</th>
              <th>Dose</th>
              <th>Próxima visita</th>
              <th>Recomendação</th>
              <th v-if="!props.isProducerView">Ações</th>
            </tr>
          </thead>
          <tbody>
            <tr v-if="visibleOrientacoes.length === 0">
              <td :colspan="props.isProducerView ? 5 : 6" class="empty-state">Nenhuma orientação cadastrada.</td>
            </tr>
            <tr v-for="item in visibleOrientacoes" :key="item.id">
              <td data-label="Produtor">{{ getProducerById(item.producerId)?.name || '—' }}</td>
              <td data-label="Defensivo">{{ getDefensivoById(item.defensivoId)?.nome || '—' }}</td>
              <td data-label="Dose">{{ item.dose }}</td>
              <td data-label="Próxima visita">{{ formatDate(item.proximaVisita) }}</td>
              <td data-label="Recomendação">{{ item.recomendacao }}</td>
              <td v-if="!props.isProducerView" class="actions-cell" data-label="Ações">
                <button type="button" class="icon-btn" @click="startEdit(item)">Editar</button>
                <button type="button" class="icon-btn danger" @click="deleteOrientacao(item.id)">Excluir</button>
              </td>
            </tr>
          </tbody>
        </table>
      </section>
    </div>

    <section v-if="!props.isProducerView" class="orientation-admin card">
      <h3>Produtores e lavouras</h3>

      <div v-if="producerList.length === 0" class="empty-box">Nenhum produtor cadastrado.</div>

      <div v-for="producer in producerList" :key="producer.id" class="producer-card">
        <div class="producer-header">
          <div>
            <h4>{{ producer.name }}</h4>
            <p>{{ producer.email }}</p>
          </div>
          <span class="badge">{{ getProducerLavouras(producer.id).length }} lavoura(s)</span>
        </div>

        <div class="producer-farms">
          <div v-if="getProducerLavouras(producer.id).length === 0" class="empty-box small">Nenhuma lavoura cadastrada.</div>

          <div v-for="lavoura in getProducerLavouras(producer.id)" :key="lavoura.id" class="farm-item">
            <div class="farm-plot" aria-label="Representação gráfica da lavoura">
              <svg width="120" height="80" viewBox="0 0 120 80" preserveAspectRatio="xMidYMid meet">
                <rect x="8" y="8" width="104" height="64" rx="6" fill="#eaf7eb" stroke="#2b8a3e" stroke-width="2" />
                <rect x="14" y="14" width="92" height="52" rx="4" fill="#d7f0d6" stroke="#2b8a3e" stroke-width="1.5" />
              </svg>
            </div>

            <div class="farm-details">
              <strong>{{ lavoura.cultura }}</strong>
              <span>Área: {{ lavoura.area || '—' }} ha</span>
              <span>Talhão: {{ lavoura.identificacao || '—' }}</span>
              <span>Plantio: {{ lavoura.dataPlantio || '—' }}</span>
            </div>
          </div>
        </div>
      </div>
    </section>
  </div>
</template>

<style scoped>
.card {
  background: #fff;
  border-radius: 10px;
  box-shadow: 0 10px 30px rgba(11, 22, 11, 0.06);
  padding: 16px;
}

.orientation-shell {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.orientation-dashboard {
  display: grid;
  grid-template-columns: 1.5fr repeat(3, minmax(0, 1fr));
  gap: 16px;
}

.dashboard-card {
  background: linear-gradient(180deg, #ffffff, #f7faf7);
  border: 1px solid #edf2ed;
  border-radius: 14px;
  padding: 18px 20px;
  box-shadow: 0 8px 22px rgba(11, 22, 11, 0.04);
}

.dashboard-highlight {
  background: linear-gradient(135deg, #eaf8ed 0%, #dff4e6 100%);
  border-color: #bee4c7;
}

.dashboard-label {
  display: inline-block;
  font-size: 12px;
  text-transform: uppercase;
  letter-spacing: 0.08em;
  color: #56705c;
  font-weight: 700;
  margin-bottom: 8px;
}

.dashboard-card h3 {
  margin: 0;
  font-size: 28px;
  color: #1e2a1d;
}

.next-visit-detail {
  margin-top: 10px;
  display: flex;
  align-items: center;
  gap: 8px;
  color: #35533c;
  font-size: 14px;
  flex-wrap: wrap;
}

.next-visit-meta {
  margin-top: 12px;
  display: inline-flex;
  padding: 6px 10px;
  border-radius: 999px;
  background: rgba(38, 132, 75, 0.12);
  color: #176831;
  font-weight: 700;
  font-size: 12px;
}

.chart-panel {
  padding: 18px 20px;
}

.chart-header {
  margin-bottom: 16px;
}

.chart-header h3 {
  margin: 0;
  font-size: 20px;
  color: #1e2a1d;
}

.chart-header p {
  margin: 6px 0 0;
  color: #5b655d;
  font-size: 14px;
}

.chart-bars {
  display: grid;
  grid-template-columns: repeat(4, minmax(0, 1fr));
  gap: 18px;
  align-items: end;
  height: 160px;
  padding: 12px 8px 0;
}

.chart-column {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-end;
  gap: 8px;
  height: 100%;
}

.chart-value {
  font-size: 12px;
  color: #2e8b57;
  font-weight: 700;
}

.chart-bar {
  width: 100%;
  max-width: 52px;
  min-height: 12px;
  border-radius: 12px 12px 0 0;
  background: linear-gradient(180deg, #79d18f 0%, #2d9a57 100%);
  box-shadow: 0 10px 18px rgba(45, 154, 87, 0.15);
}

.chart-column small {
  color: #57655d;
  font-size: 11px;
  text-transform: capitalize;
}

.orientation-top-row {
  display: grid;
  grid-template-columns: 1fr;
  gap: 20px;
  align-items: start;
}

.orientation-admin,
.orientation-form,
.orientation-history {
  width: 100%;
  border: 1px solid #edf2ed;
  background: #ffffff;
}

.orientation-history {
  grid-column: 1 / -1;
}

.orientation-admin {
  margin-top: 4px;
}

.orientation-admin h3,
.orientation-form h3,
.orientation-history h3 {
  margin: 0 0 14px;
  font-size: 20px;
  color: #1e2a1d;
  font-weight: 700;
}

.producer-card {
  border: 1px solid #ebf1eb;
  border-radius: 12px;
  padding: 14px;
  margin-bottom: 12px;
  background: linear-gradient(180deg, #f9fff9, #f3faf3);
}

.producer-header {
  display: flex;
  justify-content: space-between;
  gap: 12px;
  align-items: center;
  margin-bottom: 12px;
}

.producer-header h4 {
  margin: 0;
  font-size: 18px;
  color: #1f2d20;
}

.producer-header p {
  margin: 4px 0 0;
  font-size: 13px;
  color: #5e675f;
}

.badge {
  display: inline-flex;
  align-items: center;
  background: #e7f7ea;
  color: #1f7a35;
  border-radius: 999px;
  padding: 6px 10px;
  font-size: 12px;
  font-weight: 700;
}

.producer-farms {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.farm-item {
  display: flex;
  gap: 10px;
  align-items: center;
  border: 1px solid #e7efe7;
  border-radius: 10px;
  padding: 10px;
  background: #fff;
}

.farm-plot {
  width: 120px;
  height: 80px;
  background: #f8faf8;
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  border: 1px solid #e5f0e5;
}

.farm-details {
  display: flex;
  flex-direction: column;
  gap: 4px;
  font-size: 13px;
  color: #4a554b;
}

.nice-form {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.field {
  display: flex;
  flex-direction: column;
  gap: 6px;
  font-size: 14px;
  color: #2b352d;
}

.field input,
.field select,
.field textarea {
  width: 100%;
  border: 1px solid #e6ece7;
  border-radius: 10px;
  background: #fff;
  color: #1d261f;
  padding: 10px 12px;
  font-size: 14px;
  resize: vertical;
}

.field input:focus,
.field select:focus,
.field textarea:focus {
  outline: none;
  border-color: #2e8b57;
  box-shadow: 0 0 0 3px rgba(46, 139, 87, 0.12);
}

.grid-row {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 12px;
}

.form-actions {
  display: flex;
  gap: 10px;
  align-items: center;
  margin-top: 4px;
}

.primary-btn,
.ghost-btn,
.icon-btn {
  border-radius: 10px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.primary-btn {
  border: none;
  background: linear-gradient(180deg, #2d9a57, #228349);
  color: white;
  padding: 10px 16px;
  font-weight: 700;
}

.ghost-btn {
  background: transparent;
  border: 1px solid #dfe9df;
  color: #2d3a2e;
  padding: 9px 14px;
}

.data-table {
  width: 100%;
  border-collapse: collapse;
  table-layout: fixed;
}

.data-table th,
.data-table td {
  padding: 10px 8px;
  border-bottom: 1px solid #edf1ef;
  text-align: left;
  vertical-align: top;
  word-break: break-word;
}

.data-table th {
  font-size: 12px;
  text-transform: uppercase;
  color: #5f6f62;
  letter-spacing: 0.05em;
}

.actions-cell {
  white-space: nowrap;
}

.icon-btn {
  border: 1px solid #e4ebe5;
  background: #fff;
  color: #1f2a22;
  padding: 7px 10px;
  margin-right: 6px;
}

.icon-btn.danger {
  background: #fff2f2;
  border-color: #f4d4d4;
  color: #a33a3a;
}

.empty-box,
.empty-state {
  text-align: center;
  color: #657168;
  padding: 16px;
  background: #fafcfa;
  border-radius: 10px;
  border: 1px dashed #dfe8df;
}

.empty-box.small {
  padding: 12px;
  margin-top: 8px;
}

@media (max-width: 1100px) {
  .orientation-dashboard {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }

  .orientation-top-row {
    grid-template-columns: 1fr;
  }
}

@media (max-width: 700px) {
  .orientation-dashboard {
    grid-template-columns: 1fr;
  }

  .dashboard-card h3 {
    font-size: 22px;
  }

  .grid-row {
    grid-template-columns: 1fr;
  }

  .data-table {
    display: block;
    width: 100%;
    overflow-x: auto;
    white-space: normal;
  }

  .data-table thead,
  .data-table tbody,
  .data-table tr,
  .data-table th,
  .data-table td {
    display: block;
  }

  .data-table thead {
    display: none;
  }

  .data-table tbody tr {
    border: 1px solid #edf1ef;
    border-radius: 10px;
    padding: 10px;
    margin-bottom: 12px;
    background: #fafcfa;
  }

  .data-table td {
    border: none;
    padding: 6px 0;
    display: flex;
    justify-content: space-between;
    gap: 12px;
    font-size: 13px;
  }

  .data-table td::before {
    content: attr(data-label);
    font-weight: 700;
    color: #2d3a2e;
  }

  .data-table td:first-child {
    padding-top: 0;
  }

  .data-table td:last-child {
    padding-bottom: 0;
  }
}
</style>
