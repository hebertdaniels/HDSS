<template>
  <!--
    ESCOPO DOS SELETORES — a mesma regra do PlaygroundLayout e do
    TestDataTableClaudeCode: data-theme na raiz; data-brand só no CONTEÚDO
    (.cp-page). A barra do sandbox fica fora do escopo de marca.

    O conteúdo segue a casca dos patterns: DssAppBar + DssPageShell, que
    estabelece as zonas (casca segue a marca; conteúdo mantém o semântico).
    As coleções NÃO recebem `brand`: com a prop, viravam zona explícita e o
    "Incluir" e o "Salvar" passavam a ser cor de produto.
  -->
  <div class="cp-test" :data-theme="theme">
    <div class="cp-devbar" role="group" aria-label="Sandbox — estado das coleções">
      <div class="cp-devbar__states">
        <span class="cp-devbar__label">sandbox · estado</span>
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
      <div class="cp-devbar__presentation" role="group" aria-label="Apresentação da demonstração">
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

    <div class="cp-page" :data-brand="brand">
      <DssLayout view="hHh lpR fFf" container class="cp-layout">
        <DssAppBar :brand="brand" title="Cadastro de clientes" menu-aria-label="Abrir menu principal">
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
                  <DssBreadcrumbsEl label="Comercial" icon="looks_one" />
                  <DssBreadcrumbsEl label="Cadastro de clientes" icon="looks_two" />
                </DssBreadcrumbs>
              </template>

              <DssSectionTitle :level="1" size="lg" label="DssComposition · DssCompositionGroup" />

              <!-- ── Cenário principal: cadastro mestre com três coleções em abas ── -->
              <DssCard variant="outlined" class="cp-section" role="region" aria-labelledby="cp-mestre">
                <DssSectionTitle id="cp-mestre" :level="2" label="Cadastro mestre" />
                <DssForm ref="mestre" class="cp-form" @submit="aoSalvarCadastro" @validation-error="aoReprovar">
                  <DssInput
                    v-model="cliente.nome"
                    label="Nome do cliente"
                    :rules="[(v) => !!v || 'Informe o nome do cliente']"
                    :disabled="cenario === 'disabled'"
                  />

                  <DssCompositionGroup v-model="abaAtiva" aria-label="Coleções do cliente">
                    <DssComposition
                      v-model="contatos"
                      name="contatos"
                      label="Contatos"
                      row-key="id"
                      :columns="colunasDeContato"
                      :row-actions="acoesDeContato"
                      :new-record="novoContato"
                      v-bind="estadoComum"
                      @inactivate="aoInativar"
                      @activate="aoAtivar"
                    >
                      <template #form="{ record }">
                        <DssInput v-model="record.nome" label="Nome" :rules="[(v) => !!v || 'Informe o nome']" />
                        <DssInput v-model="record.telefone" label="Telefone" />
                      </template>
                    </DssComposition>

                    <DssComposition
                      v-model="enderecos"
                      name="enderecos"
                      label="Endereços"
                      row-key="id"
                      required
                      required-message="Inclua ao menos um endereço."
                      :columns="colunasDeEndereco"
                      :new-record="novoEndereco"
                      v-bind="estadoComum"
                    >
                      <template #form="{ record }">
                        <DssInput v-model="record.logradouro" label="Logradouro" :rules="[(v) => !!v || 'Informe o logradouro']" />
                        <DssInput v-model="record.bairro" label="Bairro" />
                      </template>
                    </DssComposition>

                    <DssComposition
                      v-model="medidores"
                      name="medidores"
                      label="Medidores"
                      row-key="id"
                      :columns="colunasDeMedidor"
                      :new-record="novoMedidor"
                      v-bind="estadoComum"
                    >
                      <template #form="{ record }">
                        <DssInput v-model="record.numero" label="Número do medidor" :rules="[(v) => !!v || 'Informe o número']" />
                      </template>
                    </DssComposition>
                  </DssCompositionGroup>

                  <div class="cp-actions">
                    <!-- ui-rules 02 §footer: à direita, secundária → primária. -->
                    <DssButton label="Limpar validação" variant="flat" @click="mestre?.resetValidation()" />
                    <DssButton label="Salvar cadastro" color="primary" type="submit" :disabled="cenario === 'disabled'" />
                  </div>
                </DssForm>
              </DssCard>

              <!-- ── Coleção sozinha: um campo comum, sem o grupo ── -->
              <DssCard variant="outlined" class="cp-section" role="region" aria-labelledby="cp-sozinha">
                <DssSectionTitle id="cp-sozinha" :level="2" label="Coleção sozinha" />
                <p class="cp-nota">Sem o grupo, a coleção é um campo comum de formulário.</p>
                <DssComposition
                  v-model="dependentes"
                  label="Dependentes"
                  row-key="id"
                  :columns="colunasDeDependente"
                  :new-record="novoDependente"
                >
                  <template #form="{ record }">
                    <DssInput v-model="record.nome" label="Nome" :rules="[(v) => !!v || 'Informe o nome']" />
                  </template>
                </DssComposition>
              </DssCard>

              <DssBanner v-if="eco" :variant="ecoTom" rounded dense dismissible @dismiss="eco = ''">{{ eco }}</DssBanner>
            </DssPageShell>
          </DssPage>
        </DssPageContainer>
      </DssLayout>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed, ref } from 'vue'
import DssComposition from '@components/composed/DssComposition/DssComposition.vue'
import DssCompositionGroup from '@components/composed/DssCompositionGroup/DssCompositionGroup.vue'
import DssForm from '@components/composed/DssForm/DssForm.vue'
import DssInput from '@components/base/DssInput/DssInput.vue'
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
import DssBanner from '@components/base/DssBanner/DssBanner.vue'

type Linha = Record<string, unknown>

const cenario = ref('normal')
const theme = ref('light')
const brand = ref('water')
const abaAtiva = ref<string | null>(null)
const eco = ref('')
const ecoTom = ref<'info' | 'success' | 'error'>('info')
const mestre = ref<{ resetValidation: () => void } | null>(null)

const modulos = [
  { icone: 'public', label: 'Mapa', ativo: false },
  { icone: 'description', label: 'Solicitações', ativo: false },
  { icone: 'shopping_cart', label: 'Comercial', ativo: true },
  { icone: 'paid', label: 'Financeiro', ativo: false },
  { icone: 'bar_chart', label: 'Relatórios', ativo: false },
]

const cenarios = [
  { value: 'normal', label: 'Normal' },
  { value: 'loading', label: 'Carregando' },
  { value: 'error', label: 'Erro de carga' },
  { value: 'disabled', label: 'Desabilitado' },
  { value: 'readonly', label: 'Somente leitura' },
]

/** O estado da barra do sandbox, aplicado às três coleções do cadastro.
 *  Sem `brand`: a marca chega pela casca (zona), não pela prop. */
const estadoComum = computed(() => ({
  loading: cenario.value === 'loading',
  error: cenario.value === 'error' ? 'O serviço do cadastro não respondeu. Tente novamente.' : null,
  disabled: cenario.value === 'disabled',
  readonly: cenario.value === 'readonly',
}))

const cliente = ref({ nome: 'Condomínio Jardim das Águas' })

// ── Contatos: registros GRAVADOS se inativam; NOVOS se excluem (decisão C/D)
const contatos = ref<Linha[]>([
  { id: 1, nome: 'Ana Souza', telefone: '(11) 3333-1111', gravado: true, ativo: true },
  { id: 2, nome: 'Bruno Lima', telefone: '(11) 3333-2222', gravado: true, ativo: false },
])
const colunasDeContato = [
  { name: 'nome', label: 'Nome', field: 'nome', align: 'left' as const },
  { name: 'telefone', label: 'Telefone', field: 'telefone', align: 'left' as const },
  { name: 'situacao', label: 'Situação', field: (l: Linha) => (l.gravado ? (l.ativo ? 'Ativo' : 'Inativo') : 'Novo'), align: 'left' as const },
]
const editar = { name: 'edit', label: 'Editar', icon: 'edit', intent: 'edit' as const }
const excluir = { name: 'remove', label: 'Excluir', icon: 'delete', intent: 'remove' as const, color: 'negative' as const }
const inativar = { name: 'inactivate', label: 'Inativar', icon: 'block', intent: 'inactivate' as const }
const ativar = { name: 'activate', label: 'Ativar', icon: 'check_circle', intent: 'activate' as const }
function acoesDeContato(linha: Linha) {
  if (!linha.gravado) return [editar, excluir]
  return linha.ativo ? [editar, inativar] : [ativar]
}
let proximoId = 100
const novoContato = () => ({ id: ++proximoId, nome: '', telefone: '', gravado: false, ativo: true })

function mudarSituacao(linha: Linha | undefined, ativo: boolean) {
  if (!linha) return
  contatos.value = contatos.value.map((c) => (c.id === linha.id ? { ...c, ativo } : c))
}
function aoInativar(payload: { row?: Linha }) {
  mudarSituacao(payload.row, false)
  avisar(`Contato ${String(payload.row?.nome)} inativado — quem mudou o registro foi a página.`, 'info')
}
function aoAtivar(payload: { row?: Linha }) {
  mudarSituacao(payload.row, true)
  avisar(`Contato ${String(payload.row?.nome)} ativado.`, 'info')
}

// ── Endereços: OBRIGATÓRIA e vazia — o caso do risco 5
const enderecos = ref<Linha[]>([])
const colunasDeEndereco = [
  { name: 'logradouro', label: 'Logradouro', field: 'logradouro', align: 'left' as const },
  { name: 'bairro', label: 'Bairro', field: 'bairro', align: 'left' as const },
]
const novoEndereco = () => ({ id: ++proximoId, logradouro: '', bairro: '' })

// ── Medidores
const medidores = ref<Linha[]>([{ id: 1, numero: 'A23-004512' }])
const colunasDeMedidor = [{ name: 'numero', label: 'Número', field: 'numero', align: 'left' as const }]
const novoMedidor = () => ({ id: ++proximoId, numero: '' })

// ── Coleção sozinha
const dependentes = ref<Linha[]>([])
const colunasDeDependente = [{ name: 'nome', label: 'Nome', field: 'nome', align: 'left' as const }]
const novoDependente = () => ({ id: ++proximoId, nome: '' })

function aplicarCenario(valor: string) {
  cenario.value = valor
  eco.value = ''
}

function avisar(texto: string, tom: 'info' | 'success' | 'error') {
  eco.value = texto
  ecoTom.value = tom
}

function aoSalvarCadastro() {
  avisar(`Cadastro válido: ${contatos.value.length} contato(s), ${enderecos.value.length} endereço(s), ${medidores.value.length} medidor(es).`, 'success')
}

function aoReprovar() {
  avisar('O cadastro tem pendências — veja as abas marcadas.', 'error')
}
</script>

<style scoped>
.cp-test{display:flex;flex-direction:column;height:100%;color:var(--dss-text-body);font-family:var(--dss-font-family-sans);background:var(--dss-surface-default)}
.cp-devbar{flex-shrink:0;display:flex;align-items:center;flex-wrap:wrap;gap:var(--dss-spacing-2);padding:var(--dss-spacing-1) var(--dss-spacing-5);background:var(--dss-surface-subtle);border-bottom:var(--dss-border-width-thin) dashed var(--dss-border-default)}
.cp-devbar__states,.cp-devbar__presentation{display:flex;align-items:center;flex-wrap:wrap;gap:var(--dss-spacing-2)}
.cp-devbar__states{flex:1;min-width:0}
.cp-devbar__presentation{margin-inline-start:auto;flex-shrink:0}
.cp-devbar__label{font-size:var(--dss-font-size-xs);text-transform:uppercase;font-weight:var(--dss-font-weight-semibold)}
.cp-page{flex:1;min-height:0;display:flex;flex-direction:column;font-size:var(--dss-font-size-sm)}
.cp-layout{flex:1;min-height:0}
.cp-section{display:flex;flex-direction:column;gap:var(--dss-spacing-3);padding:var(--dss-spacing-3);background:var(--dss-surface-default)}
.cp-form{display:flex;flex-direction:column;gap:var(--dss-spacing-4)}
.cp-actions{display:flex;flex-wrap:wrap;justify-content:flex-end;gap:var(--dss-spacing-2)}
.cp-nota{margin:0;color:var(--dss-text-subtle)}
</style>
