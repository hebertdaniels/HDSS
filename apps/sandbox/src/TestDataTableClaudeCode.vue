<template>
  <!--
    ESCOPO DOS SELETORES — a mesma regra do PlaygroundLayout:
      data-theme  na raiz: escurecer é preferência da PÁGINA inteira.
      data-brand  só no CONTEÚDO (o .cc-page). A barra do sandbox é chrome e
                  fica fora; dentro do escopo, escolher Hub pintava de laranja
                  também os botões Water e Waste do seletor.

    O CONTEÚDO segue a casca dos patterns (Solicitações, Check-in NFAg,
    Gridmaster): DssAppBar + DssPageShell. É o shell que estabelece as zonas de
    marca — trilho e app bar são CASCA (seguem a marca), o board é CONTEÚDO
    (ações voltam ao semântico). Por isso a tabela NÃO recebe `brand`: com a
    prop, ela viraria zona explícita e os botões primary passariam a ser cor de
    produto, perdendo o peso de primary/secondary/negative.
  -->
  <div class="cc-test" :data-theme="theme">
    <div class="cc-devbar" role="group" aria-label="Sandbox — estado da tabela">
      <div class="cc-devbar__states">
        <span class="cc-devbar__label">sandbox · estado</span>
        <DssButton
          v-for="item in cenarios"
          :key="item.value"
          :label="item.label"
          size="xs"
          :variant="cenario === item.value ? 'unelevated' : 'flat'"
          color="primary"
          :aria-pressed="cenario === item.value"
          @click="aplicarCenario(item.value)"
        />
      </div>
      <div class="cc-devbar__presentation" role="group" aria-label="Apresentação da demonstração">
        <DssButton
          variant="flat"
          round
          size="md"
          :icon="theme === 'dark' ? 'light_mode' : 'dark_mode'"
          :aria-label="theme === 'dark' ? 'Ativar modo claro' : 'Ativar modo escuro'"
          @click="theme = theme === 'dark' ? 'light' : 'dark'"
        />
        <PgBrandPills v-model="brand" aria-label="Marca da demonstração" />
      </div>
    </div>

    <div class="cc-page" :data-brand="brand">
      <DssLayout view="hHh lpR fFf" container class="cc-layout">
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

              <div class="cc-head">
                <div class="cc-head__texto">
                  <DssSectionTitle :level="1" size="lg" label="DssDataTable - Claude Code" />
                  <p class="cc-nota">
                    Segunda leitura do <code>jtech-data-table</code>, montada a partir da UI consolidada
                    nos patterns “Check-in NFAg - Paralelo” e “Solicitações”.
                  </p>
                </div>
                <DssChip
                  :label="modo === 'remote' ? 'Dados remotos' : 'Dados locais'"
                  color="info"
                  variant="outline"
                  icon="database"
                />
              </div>

              <!-- O FILTRO É DA PÁGINA. O bloco só exibe o chip do que está aplicado. -->
              <div class="cc-filtro" role="group" aria-label="Filtro da página">
                <DssButton
                  v-for="opcao in filtros"
                  :key="opcao.value"
                  :label="opcao.label"
                  size="sm"
                  :variant="filtro === opcao.value ? 'unelevated' : 'outline'"
                  :aria-pressed="filtro === opcao.value"
                  @click="filtro = opcao.value"
                />
              </div>

              <DssCard variant="outlined" class="cc-bloco">
                <DssDataTableClaudeCode
                  v-model="selecionadas"
                  v-model:pagination="paginacao"
                  v-model:expanded-keys="expandidas"
                  :rows="linhasVisiveis"
                  :columns="colunas"
                  :total="todasAsLinhas.length"
                  :mode="modo"
                  :loading="cenario === 'loading' || carregandoRemoto"
                  :error="cenario === 'error' ? 'O serviço de solicitações não respondeu. Tente novamente.' : null"
                  :filter-label="rotuloDoFiltro"
                  :actions="acoes"
                  :row-actions="acoesDeLinha"
                  status-column="situacao"
                  :status-map="mapaDeSituacao"
                  title="Solicitações operacionais"
                  aria-label="Solicitações operacionais"
                  selection="multiple"
                  expandable
                  @request="aoSolicitar"
                  @action="aoAgir"
                  @expand-request="aoExpandir"
                  @clear-filter="filtro = 'todas'"
                  @retry="eco = 'Nova tentativa solicitada.'"
                >
                  <template #expanded-row="{ row }">
                    <div class="cc-detail">
                      <div><span>Descrição</span><strong>{{ row.descricao }}</strong></div>
                      <div><span>Unidade</span><strong>{{ row.unidade }}</strong></div>
                      <div><span>Atualização</span><strong>{{ row.atualizacao }}</strong></div>
                    </div>
                  </template>
                </DssDataTableClaudeCode>
              </DssCard>

              <DssBanner v-if="eco" variant="info" rounded dense dismissible @dismiss="eco = ''">{{ eco }}</DssBanner>
            </DssPageShell>
          </DssPage>
        </DssPageContainer>
      </DssLayout>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed, ref } from 'vue'
import DssDataTableClaudeCode from '@components/composed/DssDataTableClaudeCode/DssDataTableClaudeCode.vue'
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

const cenario = ref('local')
const theme = ref('light')
const brand = ref('water')
const modo = ref<'local' | 'remote'>('local')
const filtro = ref('todas')
const selecionadas = ref<Record<string, unknown>[]>([])
const expandidas = ref<Array<string | number>>([])
const paginacao = ref({ page: 1, rowsPerPage: 10, rowsNumber: 5 })
const carregandoRemoto = ref(false)
const eco = ref('')

const modulos = [
  { icone: 'public', label: 'Mapa', ativo: false },
  { icone: 'description', label: 'Solicitações', ativo: true },
  { icone: 'shopping_cart', label: 'Comercial', ativo: false },
  { icone: 'paid', label: 'Financeiro', ativo: false },
  { icone: 'bar_chart', label: 'Relatórios', ativo: false },
]

const cenarios = [
  { value: 'local', label: 'Local' },
  { value: 'remote', label: 'Remoto' },
  { value: 'loading', label: 'Carregando' },
  { value: 'empty', label: 'Vazio' },
  { value: 'error', label: 'Erro' },
  { value: 'selected', label: 'Selecionado' },
]

const filtros = [
  { value: 'todas', label: 'Todas' },
  { value: 'Pendente', label: 'Pendentes' },
  { value: 'Em análise', label: 'Em análise' },
  { value: 'Concluída', label: 'Concluídas' },
]

const colunas = [
  { name: 'protocolo', label: 'Protocolo', field: 'protocolo', align: 'left' as const, sortable: true },
  { name: 'servico', label: 'Serviço', field: 'servico', align: 'left' as const, sortable: true },
  { name: 'unidade', label: 'Unidade', field: 'unidade', align: 'left' as const },
  { name: 'situacao', label: 'Situação', field: 'situacao', align: 'left' as const, sortable: true },
]

const mapaDeSituacao = {
  Pendente: { tone: 'warning' as const, icon: 'schedule' },
  'Em análise': { tone: 'info' as const, icon: 'hourglass_top' },
  Concluída: { tone: 'success' as const, icon: 'check_circle' },
  Programada: { tone: 'info' as const, icon: 'event_available' },
}

const todasAsLinhas = [
  { id: 1, protocolo: '65665262', servico: 'Religação de água', unidade: 'Centro', situacao: 'Pendente', descricao: 'Visita técnica aguardando agendamento.', atualizacao: 'Hoje, 09:42' },
  { id: 2, protocolo: '65665263', servico: 'Aferição de hidrômetro', unidade: 'Norte', situacao: 'Em análise', descricao: 'Documentação recebida para análise.', atualizacao: 'Hoje, 08:16' },
  { id: 3, protocolo: '65665264', servico: 'Taxa de vistoria', unidade: 'Sul', situacao: 'Concluída', descricao: 'Vistoria concluída e laudo emitido.', atualizacao: 'Ontem, 17:30' },
  { id: 4, protocolo: '65665265', servico: 'Corte a pedido', unidade: 'Leste', situacao: 'Pendente', descricao: 'Solicitação aguardando validação cadastral.', atualizacao: 'Ontem, 14:05' },
  { id: 5, protocolo: '65665266', servico: 'Troca de hidrômetro', unidade: 'Centro', situacao: 'Programada', descricao: 'Equipe CL251 programada para atendimento.', atualizacao: '12 out, 16:22' },
]

const linhasRemotas = ref([...todasAsLinhas].reverse())

const rotuloDoFiltro = computed(() =>
  filtro.value === 'todas' ? null : `Situação: ${filtro.value}`,
)

const linhasVisiveis = computed(() => {
  if (cenario.value === 'empty') return []
  const base = modo.value === 'remote' ? linhasRemotas.value : todasAsLinhas
  return filtro.value === 'todas' ? base : base.filter((linha) => linha.situacao === filtro.value)
})

// As ações em lote exercitam os quatro `mode` herdados do Jtech.
const acoes = [
  { name: 'nova', label: 'Nova solicitação', icon: 'add', mode: 'none' as const },
  { name: 'editar', label: 'Editar', icon: 'edit', mode: 'single' as const },
  { name: 'atribuir', label: 'Atribuir', icon: 'person_add', mode: 'multi' as const },
  { name: 'exportar', label: 'Exportar', icon: 'download', mode: 'any' as const },
]
const acoesDeLinha = [{ name: 'abrir', label: 'Abrir', icon: 'visibility' }]

function aplicarCenario(value: string) {
  cenario.value = value
  modo.value = value === 'remote' ? 'remote' : 'local'
  selecionadas.value = value === 'selected' ? [todasAsLinhas[0], todasAsLinhas[1]] : []
  expandidas.value = []
  eco.value = ''
}

function aoSolicitar(request: { pagination: typeof paginacao.value }) {
  if (modo.value !== 'remote') return
  carregandoRemoto.value = true
  window.setTimeout(() => {
    paginacao.value = { ...request.pagination, rowsNumber: linhasRemotas.value.length }
    carregandoRemoto.value = false
    eco.value = `Leitura remota simulada concluída: ${linhasRemotas.value.length} registros.`
  }, 350)
}

function aoAgir(payload: { action: { label: string }; rows: unknown[] }) {
  eco.value = `${payload.action.label}: ${payload.rows.length} registro(s).`
}

function aoExpandir(payload: { expanded: boolean; key: string | number }) {
  if (payload.expanded) eco.value = `Detalhes do registro ${payload.key} carregados.`
}
</script>

<style scoped>
.cc-test{display:flex;flex-direction:column;height:100%;color:var(--dss-text-body);font-family:var(--dss-font-family-sans);background:var(--dss-surface-default)}
.cc-devbar{flex-shrink:0;display:flex;align-items:center;flex-wrap:wrap;gap:var(--dss-spacing-2);padding:var(--dss-spacing-1) var(--dss-spacing-5);background:var(--dss-surface-subtle);border-bottom:var(--dss-border-width-thin) dashed var(--dss-border-default)}
.cc-devbar__states,.cc-devbar__presentation{display:flex;align-items:center;flex-wrap:wrap;gap:var(--dss-spacing-2)}
.cc-devbar__states{flex:1;min-width:0}
.cc-devbar__presentation{margin-inline-start:auto;flex-shrink:0}
.cc-devbar__label{font-size:var(--dss-font-size-xs);text-transform:uppercase;font-weight:var(--dss-font-weight-semibold)}
.cc-page{flex:1;min-height:0;display:flex;flex-direction:column;font-size:var(--dss-font-size-sm)}
.cc-layout{flex:1;min-height:0}
.cc-head{display:flex;align-items:flex-start;justify-content:space-between;flex-wrap:wrap;gap:var(--dss-spacing-3)}
.cc-head__texto{display:flex;flex-direction:column;gap:var(--dss-spacing-1);min-width:0}
.cc-nota{margin:0;color:var(--dss-text-subtle)}
.cc-filtro{display:flex;align-items:center;flex-wrap:wrap;gap:var(--dss-spacing-2)}
.cc-bloco{display:flex;flex-direction:column;padding:var(--dss-spacing-3);background:var(--dss-surface-default)}
.cc-detail{display:grid;grid-template-columns:repeat(3,minmax(0,1fr));gap:var(--dss-spacing-4)}
.cc-detail div{display:flex;flex-direction:column;gap:var(--dss-spacing-1)}
.cc-detail span{font-size:var(--dss-font-size-xs);color:var(--dss-text-subtle)}
@media (max-width: 768px){.cc-head{flex-direction:column}.cc-detail{grid-template-columns:1fr}}
</style>
