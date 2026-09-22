<script setup lang="ts">
import { computed, ref } from 'vue'

const props = defineProps<{
  modelValue: string
  options: string[]
  placeholder?: string
}>()

const emit = defineEmits<{
  (e: 'update:modelValue', value: string): void
}>()

const open = ref(false)
const highlight = ref(-1)

const filtered = computed(() => {
  const kw = props.modelValue.trim().toLowerCase()
  const list = kw
    ? props.options.filter((o) => o.toLowerCase().includes(kw))
    : props.options
  return list.slice(0, 8)
})

// With no options the dropdown never shows and this behaves as a plain input.
const showDropdown = computed(() => open.value && filtered.value.length > 0)

function onInput(event: Event) {
  emit('update:modelValue', (event.target as HTMLInputElement).value)
  open.value = true
  highlight.value = -1
}

function choose(option: string) {
  emit('update:modelValue', option)
  open.value = false
  highlight.value = -1
}

function onKeydown(event: KeyboardEvent) {
  if (!showDropdown.value) return
  if (event.key === 'ArrowDown' || event.key === 'ArrowUp') {
    event.preventDefault()
    const delta = event.key === 'ArrowDown' ? 1 : -1
    const n = filtered.value.length
    highlight.value = (highlight.value + delta + n) % n
  } else if (event.key === 'Enter' && highlight.value >= 0) {
    event.preventDefault()
    choose(filtered.value[highlight.value])
  } else if (event.key === 'Escape') {
    open.value = false
    highlight.value = -1
  }
}

function onBlur() {
  open.value = false
  highlight.value = -1
}
</script>

<template>
  <div class="autocomplete">
    <input
      :value="modelValue"
      class="input"
      :placeholder="placeholder"
      autocomplete="off"
      @input="onInput"
      @focus="open = true"
      @keydown="onKeydown"
      @blur="onBlur"
    />
    <ul v-if="showDropdown" class="autocomplete-list">
      <li
        v-for="(option, i) in filtered"
        :key="option"
        class="autocomplete-item"
        :class="{ active: i === highlight }"
        @mousedown.prevent="choose(option)"
        @mouseenter="highlight = i"
      >
        {{ option }}
      </li>
    </ul>
  </div>
</template>

<style scoped>
.autocomplete {
  position: relative;
}
.autocomplete-list {
  position: absolute;
  top: calc(100% + 4px);
  left: 0;
  right: 0;
  margin: 0;
  padding: 4px;
  list-style: none;
  background: #fff;
  border: 1px solid var(--border-color);
  border-radius: 8px;
  box-shadow: var(--shadow);
  max-height: 220px;
  overflow-y: auto;
  z-index: 20;
}
.autocomplete-item {
  padding: 8px 10px;
  border-radius: 6px;
  font-size: 14px;
  cursor: pointer;
}
.autocomplete-item:hover,
.autocomplete-item.active {
  background: var(--accent-bg);
}
</style>
