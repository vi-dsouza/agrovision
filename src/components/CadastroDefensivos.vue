<script setup>
import { reactive, ref, watch } from 'vue'

const STORAGE_KEY = 'agrovision:defensivos'

const readStoredDefensivos = () => {
  try {
    const saved = localStorage.getItem(STORAGE_KEY)
    return saved ? JSON.parse(saved) : []
  } catch {
    return []
  }
}

const defensivos = ref(readStoredDefensivos())

watch(
  defensivos,
  (value) => {
    try {
      localStorage.setItem(STORAGE_KEY, JSON.stringify(value))
    } catch {
    }
  },
  { deep: true }
)

const form = reactive({
  id: null,
  nome: '',
  orientacaoTecnica: '',
  unidadeUtilizada: '',
  vencimento: '',
  tipoManejo: '',
  orientacaoUso: '',
  observacoes: ''
})

const editingId = ref(null)

const nextId = () => Math.max(0, ...defensivos.value.map((item) => item.id || 0)) + 1

const resetForm = () => {
  form.id = null
  form.nome = ''
  form.orientacaoTecnica = ''
  form.unidadeUtilizada = ''
  form.vencimento = ''
  form.tipoManejo = ''
  form.orientacaoUso = ''
  form.observacoes = ''
  editingId.value = null
}

const startEdit = (item) => {
  editingId.value = item.id
  form.id = item.id
  form.nome = item.nome
  form.orientacaoTecnica = item.orientacaoTecnica
  form.unidadeUtilizada = item.unidadeUtilizada
  form.vencimento = item.vencimento
  form.tipoManejo = item.tipoManejo
  form.orientacaoUso = item.orientacaoUso
  form.observacoes = item.observacoes || ''
}

const submitDefensivo = () => {
  if (!form.nome || !form.orientacaoTecnica || !form.unidadeUtilizada || !form.tipoManejo || !form.orientacaoUso) {
    alert('Preencha os campos obrigatórios: nome, orientação técnica, unidade, tipo de manejo e orientação de uso.')
    return
  }

  const payload = {
    id: form.id ?? nextId(),
    nome: form.nome,
    orientacaoTecnica: form.orientacaoTecnica,
    unidadeUtilizada: form.unidadeUtilizada,
    vencimento: form.vencimento,
    tipoManejo: form.tipoManejo,
    orientacaoUso: form.orientacaoUso,
    observacoes: form.observacoes
  }

  if (editingId.value === null) {
    defensivos.value.push(payload)
  } else {
    const idx = defensivos.value.findIndex((item) => item.id === editingId.value)
    if (idx !== -1) {
      defensivos.value[idx] = payload
    }
  }

  resetForm()
}

const deleteDefensivo = (id) => {
  if (!confirm('Tem certeza que deseja excluir este defensivo?')) return
  defensivos.value = defensivos.value.filter((item) => item.id !== id)
}
</script>

<template>
  <div class="defensive-shell">
    <section class="defensive-form card">
      <h3>{{ editingId === null ? 'Cadastro de defensivo agrícola' : 'Editar defensivo agrícola' }}</h3>

      <form @submit.prevent="submitDefensivo" class="nice-form defensive-fields">
        <label class="field">
          <span>Nome do defensivo</span>
          <input v-model="form.nome" type="text" placeholder="Ex: Atrazina 500 SC" />
        </label>

        <label class="field">
          <span>Orientação técnica</span>
          <textarea v-model="form.orientacaoTecnica" rows="3" placeholder="Descreva a orientação técnica recomendada"></textarea>
        </label>

        <div class="grid-row">
          <label class="field">
            <span>Unidade utilizada</span>
            <input v-model="form.unidadeUtilizada" type="text" placeholder="Ex: L/ha, kg/ha, ml/100L" />
          </label>

          <label class="field">
            <span>Vencimento</span>
            <input v-model="form.vencimento" type="date" />
          </label>
        </div>

        <div class="grid-row">
          <label class="field">
            <span>Tipo de manejo</span>
            <select v-model="form.tipoManejo">
              <option value="">Selecione</option>
              <option value="Preventivo">Preventivo</option>
              <option value="Curativo">Curativo</option>
              <option value="Integrado">Integrado</option>
              <option value="Cultural">Cultural</option>
            </select>
          </label>

          <label class="field">
            <span>Tempo de segurança</span>
            <input v-model="form.observacoes" type="text" placeholder="Ex: 7 dias" />
          </label>
        </div>

        <label class="field">
          <span>Orientação para uso</span>
          <textarea v-model="form.orientacaoUso" rows="3" placeholder="Informe recomendação de uso, dose, forma de aplicação e cuidados"></textarea>
        </label>

        <div class="form-actions">
          <button type="submit" class="primary-btn">
            {{ editingId === null ? 'Cadastrar' : 'Salvar' }}
          </button>
          <button type="button" class="ghost-btn" @click="resetForm">Limpar</button>
        </div>
      </form>
    </section>

    <section class="defensive-table card">
      <h3>Defensivos cadastrados</h3>

      <table class="data-table">
        <thead>
          <tr>
            <th>Nome</th>
            <th>Orientação técnica</th>
            <th>Unidade</th>
            <th>Vencimento</th>
            <th>Tipo manejo</th>
            <th>Uso</th>
            <th>Ações</th>
          </tr>
        </thead>
        <tbody>
          <tr v-if="defensivos.length === 0">
            <td colspan="7" class="empty-state">Nenhum defensivo cadastrado.</td>
          </tr>
          <tr v-for="item in defensivos" :key="item.id">
            <td data-label="Nome">{{ item.nome }}</td>
            <td data-label="Orientação técnica">{{ item.orientacaoTecnica }}</td>
            <td data-label="Unidade">{{ item.unidadeUtilizada }}</td>
            <td data-label="Vencimento">{{ item.vencimento || '—' }}</td>
            <td data-label="Tipo manejo">{{ item.tipoManejo || '—' }}</td>
            <td data-label="Uso">{{ item.orientacaoUso }}</td>
            <td data-label="Ações" class="actions-cell">
              <button type="button" class="icon-btn" title="Editar" @click="startEdit(item)">Editar</button>
              <button type="button" class="icon-btn danger" title="Excluir" @click="deleteDefensivo(item.id)">Excluir</button>
            </td>
          </tr>
        </tbody>
      </table>
    </section>
  </div>
</template>

<style scoped>
.card {
  background: #fff;
  padding: 16px;
  border-radius: 10px;
  box-shadow: 0 10px 30px rgba(11, 22, 11, 0.06);
}

.defensive-shell {
  display: grid;
  grid-template-columns: minmax(320px, 420px) minmax(0, 1fr);
  gap: 20px;
  align-items: start;
}

.defensive-form h3,
.defensive-table h3 {
  margin: 0 0 14px 0;
  font-size: 20px;
  color: #2d2d2d;
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
  color: #333;
  font-size: 14px;
}

.field input,
.field select,
.field textarea {
  width: 100%;
  border: 1px solid #e6e6e6;
  border-radius: 8px;
  padding: 10px 12px;
  font-size: 14px;
  background: #fff;
  color: #222;
  resize: vertical;
}

.field input:focus,
.field select:focus,
.field textarea:focus {
  outline: none;
  border-color: #2b8a3e;
  box-shadow: 0 8px 20px rgba(43, 138, 62, 0.08);
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
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.18s ease;
}

.primary-btn {
  background: #2b8a3e;
  color: #fff;
  border: none;
  padding: 10px 16px;
  font-weight: 600;
  box-shadow: 0 8px 20px rgba(43, 138, 62, 0.08);
}

.ghost-btn {
  background: transparent;
  border: 1px solid #e6e6e6;
  color: #333;
  padding: 9px 14px;
}

.data-table {
  width: 100%;
  border-collapse: collapse;
  table-layout: fixed;
}

.data-table th,
.data-table td {
  text-align: left;
  padding: 10px 8px;
  border-bottom: 1px solid #eee;
  vertical-align: top;
  word-break: break-word;
}

.data-table th {
  font-size: 12px;
  text-transform: uppercase;
  color: #666;
}

.data-table tbody tr:hover {
  background: #fafafa;
}

.actions-cell {
  white-space: nowrap;
}

.icon-btn {
  border: 1px solid #e5e5e5;
  background: #fff;
  color: #222;
  padding: 7px 10px;
  margin-right: 6px;
}

.icon-btn.danger {
  background: #ffecec;
  border-color: #ffd6d6;
}

.empty-state {
  text-align: center;
  color: #666;
  padding: 20px;
}

@media (max-width: 1100px) {
  .defensive-shell {
    grid-template-columns: 1fr;
  }
}

@media (max-width: 700px) {
  .grid-row {
    grid-template-columns: 1fr;
  }

  .data-table,
  .data-table thead,
  .data-table tbody,
  .data-table tr,
  .data-table th,
  .data-table td {
    display: block;
    width: 100%;
  }

  .data-table thead {
    display: none;
  }

  .data-table tbody {
    display: flex;
    flex-direction: column;
    gap: 12px;
  }

  .data-table tbody tr {
    background: #fafcfb;
    border: 1px solid #ebf0eb;
    border-radius: 10px;
    padding: 12px 10px;
  }

  .data-table td {
    border: none;
    display: flex;
    justify-content: space-between;
    align-items: flex-start;
    gap: 10px;
    padding: 8px 0;
    font-size: 14px;
  }

  .data-table td::before {
    content: attr(data-label);
    font-weight: 700;
    color: #334136;
    min-width: 110px;
    flex-shrink: 0;
  }

  .data-table td:last-child {
    justify-content: flex-end;
  }
}
</style>
