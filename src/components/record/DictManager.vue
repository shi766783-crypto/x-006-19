<script setup lang="ts">
import { reactive } from 'vue'
import { useFamilyStore } from '../../stores/useFamilyStore'
import type { HospitalDepartmentDict } from '../../types'

const store = useFamilyStore()

const inputs = reactive<Record<keyof HospitalDepartmentDict, string>>({
  hospitals: '',
  departments: '',
})

const sections: {
  kind: keyof HospitalDepartmentDict
  title: string
  icon: string
  placeholder: string
  emptyText: string
}[] = [
  {
    kind: 'hospitals',
    title: '常用医院',
    icon: '🏥',
    placeholder: '输入医院名称，回车添加',
    emptyText: '暂无常用医院',
  },
  {
    kind: 'departments',
    title: '常用科室',
    icon: '🩺',
    placeholder: '输入科室名称，回车添加',
    emptyText: '暂无常用科室',
  },
]

function add(kind: keyof HospitalDepartmentDict) {
  store.addDictEntry(kind, inputs[kind])
  inputs[kind] = ''
}
</script>

<template>
  <div class="dict-manager">
    <p class="hint">维护常用医院与科室后，录入就医记录时会自动补全；也可以直接输入新名称。</p>
    <section v-for="s in sections" :key="s.kind" class="dict-section">
      <h4 class="dict-title">{{ s.icon }} {{ s.title }}</h4>
      <div class="dict-add">
        <input
          v-model="inputs[s.kind]"
          class="input"
          :placeholder="s.placeholder"
          @keydown.enter.prevent="add(s.kind)"
        />
        <button type="button" class="btn btn-primary btn-sm" @click="add(s.kind)">添加</button>
      </div>
      <div v-if="store.state.dict[s.kind].length" class="chip-list">
        <span v-for="name in store.state.dict[s.kind]" :key="name" class="chip">
          {{ name }}
          <button
            type="button"
            class="chip-remove"
            :aria-label="'删除 ' + name"
            @click="store.removeDictEntry(s.kind, name)"
          >
            ×
          </button>
        </span>
      </div>
      <p v-else class="empty-text">{{ s.emptyText }}</p>
    </section>
  </div>
</template>

<style scoped>
.hint {
  font-size: 13px;
  color: var(--text-secondary);
  margin-bottom: 16px;
}
.dict-section {
  margin-bottom: 20px;
}
.dict-title {
  margin: 0 0 8px;
  font-size: 15px;
  color: var(--primary-color);
}
.dict-add {
  display: flex;
  gap: 8px;
  margin-bottom: 10px;
}
.dict-add .btn {
  flex-shrink: 0;
}
.chip-list {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}
.chip {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  padding: 4px 8px 4px 12px;
  background: var(--accent-bg);
  color: var(--primary-color);
  border-radius: 14px;
  font-size: 13px;
}
.chip-remove {
  border: none;
  background: transparent;
  color: var(--text-secondary);
  cursor: pointer;
  font-size: 15px;
  line-height: 1;
  padding: 0 2px;
}
.chip-remove:hover {
  color: var(--danger-color);
}
.empty-text {
  font-size: 13px;
  color: var(--text-secondary);
}
</style>
