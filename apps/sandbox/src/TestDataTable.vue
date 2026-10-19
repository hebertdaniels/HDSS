<template>
  <!--
    ESCOPO DOS SELETORES — mesma regra do PlaygroundLayout: data-theme na raiz;
    data-brand só no CONTEÚDO (.dt-page). A barra do sandbox fica fora.
    O conteúdo segue a casca dos patterns (DssAppBar + DssPageShell); a tabela
    NÃO recebe `brand`, para as ações manterem o peso semântico.
  -->
  <div class="dt-test" :data-theme="theme">
    <div class="dt-devbar" role="group" aria-label="Sandbox — estado da tabela">
      <div class="dt-devbar__states">
        <span class="dt-devbar__label">sandbox · estado</span>
        <DssButton v-for="item in scenarios" :key="item.value" :label="item.label" size="xs"
          :variant="scenario === item.value ? 'unelevated' : 'flat'" color="primary"
          :aria-pressed="scenario === item.value" @click="applyScenario(item.value)" />
      </div>
      <div class="dt-devbar__presentation" role="group" aria-label="Apresentação da demonstração">
        <DssButton variant="flat" round size="md" :icon="theme === 'dark' ? 'light_mode' : 'dark_mode'"
          :aria-label="theme === 'dark' ? 'Ativar modo claro' : 'Ativar modo escuro'" @click="theme = theme === 'dark' ? 'light' : 'dark'" />
        <PgBrandPills v-model="brand" aria-label="Marca da demonstração" />
      </div>
    </div>

    <div class="dt-page" :data-brand="brand">
      <DssLayout view="hHh lpR fFf" container class="dt-layout">
        <DssAppBar :brand="brand" title="Solicitações" menu-aria-label="Abrir menu principal">
          <template #actions>
            <DssButton variant="flat" round size="md" icon="help_outline" aria-label="Ajuda" />
            <DssButton variant="flat" round size="md" icon="account_circle" aria-label="Minha conta" />
          </template>
        </DssAppBar>

        <DssPageContainer>
          <DssPage>
            <DssPageShell rail-aria-label="Módulos do sistema">
              <template #rail>
                <DssPageShellRailItem
                  v-for="m in modulos"
                  :key="m.label"
                  :icon="m.icone"
                  :label="m.label"
                  :active="m.ativo"
                />
              </template>

              <template #breadcrumb>
                <DssBreadcrumbs separator="›" gutter="sm">
                  <DssBreadcrumbsEl label="Operação" icon="looks_one" />
                  <DssBreadcrumbsEl label="Solicitações operacionais" icon="looks_two" />
                </DssBreadcrumbs>
              </template>

              <div class="dt-head">
                <div class="dt-head__texto">
                  <DssSectionTitle :level="1" size="lg" label="DssDataTable" />
                  <p class="dt-nota">Piloto operacional com API Vue 3 controlada e composição exclusiva de componentes DSS.</p>
                </div>
                <DssChip :label="mode === 'remote' ? 'Dados remotos' : 'Dados locais'" color="info" variant="outline" icon="database" />
              </div>

              <DssCard variant="outlined" class="dt-bloco">
                <DssDataTable
                  v-model="selected"
                  v-model:filter="filter"
                  v-model:pagination="pagination"
                  v-model:expanded-keys="expanded"
                  :rows="visibleRows"
                  :columns="columns"
                  :mode="mode"
                  :loading="scenario === 'loading' || remoteLoading"
                  :error="scenario === 'error' ? 'O serviço de solicitações não respondeu. Tente novamente.' : null"
                  :actions="actions"
                  :row-actions="rowActions"
                  :pagination="pagination"
                  title="Solicitações operacionais"
                  aria-label="Solicitações operacionais"
                  selection="multiple"
                  expandable
                  density="compact"
                  @request="handleRequest"
                  @action="handleAction"
                  @expand-request="handleExpand"
                >
                  <template #expanded-row="{ row }">
                    <div class="dt-detail">
                      <div><span>Descrição</span><strong>{{ row.descricao }}</strong></div>
                      <div><span>Unidade</span><strong>{{ row.unidade }}</strong></div>
                      <div><span>Atualização</span><strong>{{ row.atualizacao }}</strong></div>
                    </div>
                  </template>
                </DssDataTable>
              </DssCard>

              <DssBanner v-if="feedback" variant="info" rounded dense dismissible @dismiss="feedback = ''">{{ feedback }}</DssBanner>
            </DssPageShell>
          </DssPage>
        </DssPageContainer>
      </DssLayout>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed, ref } from 'vue'
import DssDataTable from '@components/composed/DssDataTable/DssDataTable.vue'
import DssButton from '@components/base/DssButton/DssButton.vue'
import DssAppBar from '@components/composed/DssAppBar/DssAppBar.vue'
import DssBreadcrumbs from '@components/base/DssBreadcrumbs/DssBreadcrumbs.vue'
import DssBreadcrumbsEl from '@components/base/DssBreadcrumbsEl/DssBreadcrumbsEl.vue'
import DssCard from '@components/base/DssCard/DssCard.vue'
import DssLayout from '@components/base/DssLayout/DssLayout.vue'
import DssPage from '@components/base/DssPage/DssPage.vue'
import DssPageContainer from '@components/base/DssPageContainer/DssPageContainer.vue'
import DssPageShell from '@components/composed/DssPageShell/DssPageShell.vue'
import DssPageShellRailItem from '@components/composed/DssPageShell/DssPageShellRailItem.vue'
import DssSectionTitle from '@components/base/DssSectionTitle/DssSectionTitle.vue'
import { PgBrandPills } from './playground'
import DssChip from '@components/base/DssChip/DssChip.vue'
import DssBanner from '@components/base/DssBanner/DssBanner.vue'

const scenario = ref('local')
const theme = ref('light')
const brand = ref('water')
const mode = ref<'local' | 'remote'>('local')
const filter = ref('')
const selected = ref<Record<string, unknown>[]>([])
const expanded = ref<Array<string | number>>([])
const pagination = ref({ page: 1, rowsPerPage: 5, rowsNumber: 8 })
const remoteLoading = ref(false)
const feedback = ref('')
const modulos = [
  { icone: 'public', label: 'Mapa', ativo: false },
  { icone: 'description', label: 'Solicitações', ativo: true },
  { icone: 'shopping_cart', label: 'Comercial', ativo: false },
  { icone: 'paid', label: 'Financeiro', ativo: false },
  { icone: 'bar_chart', label: 'Relatórios', ativo: false },
]
const scenarios = [
  { value: 'local', label: 'Local' }, { value: 'remote', label: 'Remoto' },
  { value: 'loading', label: 'Carregando' }, { value: 'empty', label: 'Vazio' },
  { value: 'error', label: 'Erro' }, { value: 'selected', label: 'Selecionado' },
]
const columns = [
  { name: 'protocolo', label: 'Protocolo', field: 'protocolo', align: 'left' as const, sortable: true },
  { name: 'servico', label: 'Serviço', field: 'servico', align: 'left' as const, sortable: true },
  { name: 'unidade', label: 'Unidade', field: 'unidade', align: 'left' as const },
  { name: 'situacao', label: 'Situação', field: 'situacao', align: 'left' as const, sortable: true },
]
const allRows = [
  { id: 1, protocolo: '65665262', servico: 'Religação de água', unidade: 'Centro', situacao: 'Pendente', descricao: 'Visita técnica aguardando agendamento.', atualizacao: 'Hoje, 09:42' },
  { id: 2, protocolo: '65665263', servico: 'Aferição de hidrômetro', unidade: 'Norte', situacao: 'Em análise', descricao: 'Documentação recebida para análise.', atualizacao: 'Hoje, 08:16' },
  { id: 3, protocolo: '65665264', servico: 'Taxa de vistoria', unidade: 'Sul', situacao: 'Concluída', descricao: 'Vistoria concluída e laudo emitido.', atualizacao: 'Ontem, 17:30' },
  { id: 4, protocolo: '65665265', servico: 'Corte a pedido', unidade: 'Leste', situacao: 'Pendente', descricao: 'Solicitação aguardando validação cadastral.', atualizacao: 'Ontem, 14:05' },
  { id: 5, protocolo: '65665266', servico: 'Troca de hidrômetro', unidade: 'Centro', situacao: 'Programada', descricao: 'Equipe CL251 programada para atendimento.', atualizacao: '12 out, 16:22' },
]
const remoteRows = ref([...allRows].reverse())
const visibleRows = computed(() => scenario.value === 'empty' ? [] : mode.value === 'remote' ? remoteRows.value : allRows)
const actions = [
  { name: 'assign', label: 'Atribuir', icon: 'person_add', requiresSelection: true, multiple: true },
  { name: 'export', label: 'Exportar', icon: 'download', multiple: true },
]
const rowActions = [{ name: 'open', label: 'Abrir', icon: 'visibility' }]

function applyScenario(value: string) {
  scenario.value = value
  mode.value = value === 'remote' ? 'remote' : 'local'
  selected.value = value === 'selected' ? [allRows[0], allRows[1]] : []
  expanded.value = []
  feedback.value = ''
}
function handleRequest(request: { filter?: string; pagination: typeof pagination.value }) {
  if (mode.value !== 'remote') return
  remoteLoading.value = true
  window.setTimeout(() => {
    remoteRows.value = [...allRows].filter(row => !request.filter || Object.values(row).some(value => String(value).toLowerCase().includes(String(request.filter).toLowerCase())))
    pagination.value = { ...request.pagination, rowsNumber: remoteRows.value.length }
    remoteLoading.value = false
    feedback.value = `Leitura remota simulada concluída: ${remoteRows.value.length} registros.`
  }, 350)
}
function handleAction(payload: { action: { label: string }; rows: unknown[] }) { feedback.value = `${payload.action.label}: ${payload.rows.length} registro(s).` }
function handleExpand(payload: { expanded: boolean; key: string | number }) { if (payload.expanded) feedback.value = `Detalhes do registro ${payload.key} carregados.` }
</script>

<style scoped>
.dt-test{display:flex;flex-direction:column;height:100%;color:var(--dss-text-body);font-family:var(--dss-font-family-sans);background:var(--dss-surface-default)}
.dt-devbar{flex-shrink:0;display:flex;align-items:center;flex-wrap:wrap;gap:var(--dss-spacing-2);padding:var(--dss-spacing-1) var(--dss-spacing-5);background:var(--dss-surface-subtle);border-bottom:var(--dss-border-width-thin) dashed var(--dss-border-default)}
.dt-devbar__states,.dt-devbar__presentation{display:flex;align-items:center;flex-wrap:wrap;gap:var(--dss-spacing-2)}
.dt-devbar__states{flex:1;min-width:0}.dt-devbar__presentation{margin-inline-start:auto;flex-shrink:0}
.dt-devbar__label{font-size:var(--dss-font-size-xs);text-transform:uppercase;font-weight:var(--dss-font-weight-semibold)}
.dt-page{flex:1;min-height:0;display:flex;flex-direction:column;font-size:var(--dss-font-size-sm)}
.dt-layout{flex:1;min-height:0}
.dt-head{display:flex;align-items:flex-start;justify-content:space-between;flex-wrap:wrap;gap:var(--dss-spacing-3)}
.dt-head__texto{display:flex;flex-direction:column;gap:var(--dss-spacing-1);min-width:0}
.dt-nota{margin:0;color:var(--dss-text-subtle)}
.dt-bloco{display:flex;flex-direction:column;padding:var(--dss-spacing-3);background:var(--dss-surface-default)}
.dt-detail{display:grid;grid-template-columns:repeat(3,minmax(0,1fr));gap:var(--dss-spacing-4)}
.dt-detail div{display:flex;flex-direction:column;gap:var(--dss-spacing-1)}.dt-detail span{font-size:var(--dss-font-size-xs);color:var(--dss-text-subtle)}
@media (max-width: 768px){.dt-head{flex-direction:column}.dt-detail{grid-template-columns:1fr}}
</style>
