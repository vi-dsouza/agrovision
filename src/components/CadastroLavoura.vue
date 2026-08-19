<script setup>
import { ref, reactive, computed } from 'vue'

const props = defineProps({
  producers: { type: Array, default: () => [] }
})

const emit = defineEmits(['saved'])

const form = reactive({
  id: null,
  cultura: '',
  area: '',
  largura: '',
  comprimento: '',
  dataPlantio: '',
  identificacao: '',
  produtorId: null
})

const lavouras = ref([])

const selectedLavoura = ref(null)
const modalVisible = ref(false)

const selectedOthers = computed(() => {
  if (!selectedLavoura.value) return []
  return lavouras.value.filter(l => l.produtorId === selectedLavoura.value.produtorId && l.id !== selectedLavoura.value.id)
})

const viewDetails = (l) => {
  selectedLavoura.value = l
  modalVisible.value = true
}

const closeModal = () => {
  modalVisible.value = false
}

const modalSimulationSize = computed(() => {
  const l = selectedLavoura.value
  const w = l ? (Number(l.largura) || 10) : (Number(form.largura) || 10)
  const h = l ? (Number(l.comprimento) || 10) : (Number(form.comprimento) || 10)
  return { w, h }
})

const nextId = () => Math.max(0, ...lavouras.value.map(l => l.id || 0)) + 1

const resetForm = () => {
  form.id = null
  form.cultura = ''
  form.area = ''
  form.largura = ''
  form.comprimento = ''
  form.dataPlantio = ''
  form.identificacao = ''
  form.produtorId = null
}

const startEdit = (l) => {
  form.id = l.id
  form.cultura = l.cultura
  form.area = l.area
  form.largura = l.largura
  form.comprimento = l.comprimento
  form.dataPlantio = l.dataPlantio
  form.identificacao = l.identificacao
  form.produtorId = l.produtorId
}

const submit = () => {
  if (!form.cultura || !form.produtorId) {
    alert('Preencha pelo menos a cultura e selecione um produtor.')
    return
  }
  const payload = { ...form }
  if (!form.id) {
    payload.id = nextId()
    lavouras.value.push(payload)
  } else {
    const idx = lavouras.value.findIndex(x => x.id === form.id)
    if (idx !== -1) lavouras.value[idx] = { ...payload }
  }
  resetForm()
  emit('saved')
}

const deleteLavoura = (id) => {
  if (!confirm('Excluir lavoura?')) return
  lavouras.value = lavouras.value.filter(l => l.id !== id)
}

// for simulation: compute view box size and scale
const simulationSize = computed(() => {
  const w = Number(form.largura) || 10
  const h = Number(form.comprimento) || 10
  return { w, h }
})

</script>

<template>
  <div class="farm-shell">
    <section class="farm-form card">
      <h3>Cadastro da lavoura</h3>

      <form @submit.prevent="submit" class="nice-form farm-fields">
        <label class="field">
          <span>Cultura plantada</span>
          <input class="text-input" v-model="form.cultura" placeholder="Ex: Soja" />
        </label>

        <label class="field">
          <span>Produtor</span>
          <select class="text-input" v-model.number="form.produtorId">
            <option :value="null" disabled>Selecione um produtor</option>
            <option v-for="p in props.producers" :key="p.id" :value="p.id">{{ p.name }}</option>
          </select>
        </label>

        <label class="field">
          <span>Área cultivada (ha)</span>
          <input class="text-input" v-model="form.area" type="number" step="0.01" placeholder="Área em hectares" />
        </label>

        <div class="grid-row">
          <label class="field">
            <span>Largura (m)</span>
            <input class="text-input" v-model="form.largura" type="number" step="0.1" placeholder="m" />
          </label>
          <label class="field">
            <span>Comprimento (m)</span>
            <input class="text-input" v-model="form.comprimento" type="number" step="0.1" placeholder="m" />
          </label>
        </div>

        <label class="field">
          <span>Data do plantio</span>
          <input class="text-input" v-model="form.dataPlantio" type="date" />
        </label>

        <label class="field">
          <span>Identificação do talhão</span>
          <input class="text-input" v-model="form.identificacao" placeholder="Talhão 1" />
        </label>

        <div class="actions">
          <button type="submit" class="primary">Salvar</button>
          <button type="button" class="ghost" @click="resetForm">Limpar</button>
        </div>
      </form>
    </section>

    <section class="farm-simulation card">
      <h3>Simulação da lavoura</h3>
      <div class="sim-wrap">
        <div class="sim-canvas">
          <svg v-if="form.largura || form.comprimento" :width="420" :height="260" viewBox="0 0 420 260" class="sim-svg">
            <rect x="20" y="20" :width="(Math.min(380, simulationSize.w / Math.max(simulationSize.h || 1,1) * 360))" :height="(Math.min(220, simulationSize.h / Math.max(simulationSize.w || 1,1) * 200))" fill="#e6f4ea" stroke="#2b8a3e" stroke-width="2" rx="6" />
          </svg>
          <div v-else class="sim-empty">Preencha largura/comprimento para ver a simulação</div>
        </div>

        <div class="sim-info">
          <p><strong>Cultura:</strong> {{ form.cultura || '—' }}</p>
          <p><strong>Área:</strong> {{ form.area || '—' }} ha</p>
          <p><strong>Medidas:</strong> {{ form.largura || '—' }} m × {{ form.comprimento || '—' }} m</p>
          <p><strong>Plantio:</strong> {{ form.dataPlantio || '—' }}</p>
          <p><strong>Talhão:</strong> {{ form.identificacao || '—' }}</p>
        </div>
      </div>

      <h4>Lista de lavouras</h4>
      <table class="lavouras-table">
        <thead>
          <tr><th>Produtor</th><th>Cultura</th><th>Área (ha)</th><th>Talhão</th><th></th></tr>
        </thead>
        <tbody>
          <tr v-for="l in lavouras" :key="l.id" @click="viewDetails(l)" style="cursor:pointer">
            <td>{{ (props.producers.find(p => p.id === l.produtorId) || {}).name || '—' }}</td>
            <td>{{ l.cultura }}</td>
            <td>{{ l.area }}</td>
            <td>{{ l.identificacao }}</td>
            <td>
              <button @click.stop="viewDetails(l)" class="icon-btn view">Visualizar</button>
              <button @click.stop="startEdit(l)" class="icon-btn">Editar</button>
              <button @click.stop="deleteLavoura(l.id)" class="icon-btn danger">Excluir</button>
            </td>
          </tr>
        </tbody>
      </table>

      <!-- detalhes agora apenas no modal -->
    </section>
    <!-- Modal de visualização -->
    <div v-if="modalVisible" class="modal-overlay" @click="closeModal">
      <div class="modal-content" @click.stop>
        <div class="modal-header">
          <h3>Visualizar lavoura</h3>
          <button class="ghost" @click="closeModal">Fechar</button>
        </div>
        <div class="modal-body">
          <div class="modal-sim">
            <svg v-if="selectedLavoura" :width="520" :height="320" viewBox="0 0 520 320" class="sim-svg">
              <rect x="24" y="24" :width="(Math.min(472, modalSimulationSize.w / Math.max(modalSimulationSize.h || 1,1) * 440))" :height="(Math.min(272, modalSimulationSize.h / Math.max(modalSimulationSize.w || 1,1) * 240))" fill="#e6f4ea" stroke="#2b8a3e" stroke-width="2" rx="6" />
            </svg>
            <div v-else class="sim-empty">Sem dados</div>
          </div>

          <div class="modal-info">
            <h4 class="mi-title">Informações</h4>
            <div class="mi-grid">
              <div class="mi-label">Produtor</div>
              <div class="mi-value">{{ (props.producers.find(p => p.id === selectedLavoura.produtorId) || {}).name || '—' }}</div>
              <div class="mi-label">E-mail</div>
              <div class="mi-value">{{ (props.producers.find(p => p.id === selectedLavoura.produtorId) || {}).email || '—' }}</div>
              <div class="mi-label">Cultura</div>
              <div class="mi-value">{{ selectedLavoura.cultura }}</div>
              <div class="mi-label">Área</div>
              <div class="mi-value">{{ selectedLavoura.area }} ha</div>
              <div class="mi-label">Medidas</div>
              <div class="mi-value">{{ selectedLavoura.largura }} m × {{ selectedLavoura.comprimento }} m</div>
              <div class="mi-label">Plantio</div>
              <div class="mi-value">{{ selectedLavoura.dataPlantio }}</div>
              <div class="mi-label">Identificação</div>
              <div class="mi-value">{{ selectedLavoura.identificacao }}</div>
            </div>

            <div v-if="selectedOthers.length" class="other-list">
              <h5>Outras lavouras deste produtor</h5>
              <ul>
                <li v-for="o in selectedOthers" :key="o.id">
                  <strong>{{ o.cultura }}</strong> — {{ o.area }} ha — <button class="ghost small" @click="viewDetails(o)">Ver</button>
                </li>
              </ul>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* Card and layout */
.card { background:#fff; padding:16px; border-radius:10px; box-shadow:0 10px 30px rgba(11,22,11,0.06); }
.farm-shell { display:flex; gap:18px; align-items:flex-start; flex-direction:row; flex-wrap:nowrap }
.farm-form { flex:0 0 360px; max-width:360px }

/* Form fields */
.farm-fields { display:flex; flex-direction:column; gap:12px }
.farm-fields label { display:flex; flex-direction:column; font-size:14px; color:#333; gap:6px }
.farm-fields input[type="text"],
.farm-fields input[type="number"],
.farm-fields input[type="date"],
.farm-fields select { padding:10px 12px; border-radius:8px; border:1px solid #e6e6e6; background:#fff; transition:box-shadow .18s ease, border-color .18s ease; font-size:14px }
.farm-fields input::placeholder { color:#bdbdbd }
.farm-fields input:focus, .farm-fields select:focus { outline:none; border-color:#2b8a3e; box-shadow:0 8px 20px rgba(43,138,62,0.08) }

.grid-row { display:flex; gap:10px }
.grid-row > * { flex:1; min-width:0 }

/* ensure inputs size correctly */
.farm-fields .field { width:100% }
.farm-fields .field > .text-input, .farm-fields .field > input, .farm-fields .field > select { width:100%; box-sizing:border-box }

.actions { display:flex; gap:10px; margin-top:8px }
.primary { background:#2b8a3e; color:#fff; padding:10px 14px; border-radius:10px; border:none; box-shadow:0 8px 20px rgba(43,138,62,0.08); cursor:pointer }
.primary:hover { transform:translateY(-1px) }
.ghost { background:transparent; border:1px solid #ddd; padding:10px 12px; border-radius:10px }

.text-input { padding:10px 12px; border-radius:8px; border:1px solid #e6e6e6; background:#fff; transition:box-shadow .18s ease, border-color .18s ease; font-size:14px }
.text-input::placeholder { color:#bdbdbd }
.text-input:focus { outline:none; border-color:#2b8a3e; box-shadow:0 8px 20px rgba(43,138,62,0.08) }

/* Simulation */
.farm-simulation { flex:1 }
.sim-wrap { display:flex; gap:18px; align-items:flex-start; flex-direction:row; flex-wrap:nowrap; justify-content:flex-start }
.sim-canvas { flex:1; min-width:420px }
.sim-svg { background:#f7fff7; border-radius:8px; box-shadow: inset 0 1px 0 rgba(255,255,255,0.6); width:100%; max-width:620px; height:260px }
.sim-empty { width:100%; max-width:620px; height:260px; display:flex; align-items:center; justify-content:center; color:#999; background:#fafafa; border-radius:8px }
.sim-info { background:#fff; padding:16px; border-radius:8px; box-shadow:0 6px 18px rgba(0,0,0,0.04); width:300px; flex-shrink:0 }
.sim-info p { margin:6px 0; color:#333 }

.lavouras-table { width:100%; border-collapse:collapse; margin-top:12px }
.lavouras-table th, .lavouras-table td { padding:10px 8px; border-bottom:1px solid #f0f0f0; text-align:left; font-size:14px }
.lavouras-table thead th { font-weight:600; color:#444 }

.icon-btn { padding:8px 10px; border-radius:8px; border:1px solid #e6e6e6; background:#fff; cursor:pointer }
.icon-btn:hover { box-shadow:0 6px 18px rgba(0,0,0,0.04) }
.icon-btn.danger { background:#fff0f0; border-color:#ffd6d6 }

.icon-btn.view { background:#eef9f0; border-color:#c8efce }
.ghost.small { padding:6px 8px; font-size:13px }

.detail-card { margin-top:12px; padding:12px }

/* Modal styles */
.modal-overlay { position:fixed; inset:0; background:rgba(0,0,0,0.35); display:flex; align-items:center; justify-content:center; z-index:1200 }
.modal-content { background:#fff; border-radius:10px; padding:16px; width:92%; max-width:1000px; box-shadow:0 30px 60px rgba(0,0,0,0.18) }
.modal-header { display:flex; justify-content:space-between; align-items:center; gap:12px }
.modal-body { display:flex; gap:18px; align-items:flex-start; margin-top:12px }
.modal-sim { flex:1 }
.modal-info { width:300px; flex-shrink:0; background:#fff; padding:12px; border-radius:8px; box-shadow:0 6px 18px rgba(0,0,0,0.04) }
.modal-info p { margin:6px 0 }

.mi-title { margin:0 0 8px 0; font-size:16px }
.mi-grid { display:grid; grid-template-columns: 1fr 1fr; gap:8px 12px; align-items:start }
.mi-label { color:#666; font-size:13px }
.mi-value { font-weight:600; color:#222 }
.other-list { margin-top:12px }
.other-list ul { padding-left:16px; margin:6px 0 }
.other-list li { margin:6px 0 }

.modal-header h3 { margin:0; font-size:18px }
.modal-content { padding:20px }

@media (max-width: 900px) {
  .modal-body { flex-direction:column }
  .modal-info { width:100% }
}

@media (max-width: 900px) {
  .farm-shell { flex-direction:column }
  .farm-form { width:100%; max-width:100% }
  .sim-wrap { flex-direction:column }
  .sim-canvas { min-width:100%; flex:1 }
  .sim-empty, .sim-svg { width:100%; height:200px }
  .sim-info { width:100%; }
}
</style>
