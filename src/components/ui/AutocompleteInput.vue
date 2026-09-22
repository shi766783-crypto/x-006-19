<script setup lang="ts">
import { computed, ref } from 'vue'

/**
 * Text input with optional autocomplete suggestions.
 * Never restricts input: any free-form value is allowed, suggestions are
 * only a convenience. With an empty suggestion list it behaves exactly
 * like a plain input.
 */
const props = withDefaults(
  defineProps<{
    modelValue: string
    suggestions: string[]
    placeholder?: string
  }>(),
  { placeholder: '' },
)

const emit = defineEmits<{
  (e: 'update:modelValue', value: string): void
}>()

const MAX_VISIBLE = 8

const open = ref(false)
const activeIndex = ref(-1)

const filtered = computed(() => {
  const q = props.modelValue.trim().toLowerCase()
  const list = q
    ? props.suggestions.filter((s) => s.toLowerCase().includes(q))
    : props.suggestions
  return list.slice(0, MAX_VISIBLE)
})

const showDropdown = computed(() => open.value && filtered.value.length > 0)

function onInput(event: Event) {
  emit('update:modelValue', (event.target as HTMLInputElement).value)
  open.value = true
  activeIndex.value = -1
}

function pick(value: string) {
  emit('update:modelValue', value)
  open.value = false
  activeIndex.value = -1
}

function onKeydown(event: KeyboardEvent) {
  if (!showDropdown.value) return
  if (event.key === 'ArrowDown') {
    event.preventDefault()
    activeIndex.value = (activeIndex.value + 1) % filtered.value.length
  } else if (event.key === 'ArrowUp') {
    event.preventDefault()
    activeIndex.value =
      (activeIndex.value - 1 + filtered.value.length) % filtered.value.length
  } else if (event.key === 'Enter' && activeIndex.value >= 0) {
    event.preventDefault()
    pick(filtered.value[activeIndex.value])
  } else if (event.key === 'Escape') {
    open.value = false
    activeIndex.value = -1
  }
}

function onBlur() {
  // Delay closing so a click on a suggestion registers first.
  setTimeout(() => {
    open.value = false
    activeIndex.value = -1
  }, 150)
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
    <ul v-if="showDropdown" class="suggestion-list" role="listbox">
      <li
        v-for="(item, i) in filtered"
        :key="item"
        class="suggestion-item"
        :class="{ active: i === activeIndex }"
        role="option"
        :aria-selected="i === activeIndex"
        @mousedown.prevent="pick(item)"
      >
        {{ item }}
      </li>
    </ul>
  </div>
</template>

<style scoped>
.autocomplete {
  position: relative;
}
.suggestion-list {
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
  max-height: 240px;
  overflow-y: auto;
  z-index: 20;
}
.suggestion-item {
  padding: 8px 10px;
  border-radius: 6px;
  font-size: 14px;
  cursor: pointer;
}
.suggestion-item:hover,
.suggestion-item.active {
  background: var(--accent-bg);
}
</style>
