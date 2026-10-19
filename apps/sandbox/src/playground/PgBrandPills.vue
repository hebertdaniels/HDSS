<template>
  <!--
    Seletor de marca do SANDBOX — o mesmo do PlaygroundLayout, extraído para as
    páginas de pattern (DssPageShell) usarem também.

    Os botões são CHROME: neutros, cada um com a bolinha da PRÓPRIA marca. Eles
    nunca ficam dentro do escopo `[data-brand]` que controlam — quem aplica o
    atributo é o conteúdo da página, não a barra. Com o seletor dentro do escopo,
    escolher Hub pintava de laranja também os botões Water e Waste.
  -->
  <div class="pg-brand-pills pg-brand-switch" role="radiogroup" :aria-label="ariaLabel">
    <button
      v-for="b in opcoes"
      :key="b.value || 'neutral'"
      type="button"
      role="radio"
      :aria-checked="modelValue === b.value"
      :class="['pg-pill', `pg-pill--${b.value || 'neutral'}`, { 'is-active': modelValue === b.value }]"
      @click="emit('update:modelValue', b.value)"
    >
      <span class="pg-pill__dot" aria-hidden="true"></span>
      {{ b.label }}
    </button>
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import './playground.scss'

const props = withDefaults(defineProps<{
  modelValue: string
  /** Inclui a opção "Neutro" (sem marca). As páginas de pattern não usam. */
  neutral?: boolean
  ariaLabel?: string
}>(), {
  neutral: false,
  ariaLabel: 'Selecionar marca',
})

const emit = defineEmits<{ 'update:modelValue': [value: string] }>()

const MARCAS = [
  { label: 'Hub', value: 'hub' },
  { label: 'Water', value: 'water' },
  { label: 'Waste', value: 'waste' },
]

const opcoes = computed(() =>
  props.neutral ? [{ label: 'Neutro', value: '' }, ...MARCAS] : MARCAS,
)
</script>
