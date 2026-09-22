<script setup lang="ts">
import { reactive, ref } from 'vue'
import { useFamilyStore } from '../../stores/useFamilyStore'
import EmptyState from '../ui/EmptyState.vue'

const store = useFamilyStore()

const newHospital = ref('')
const deptInputs = reactive<Record<string, string>>({})
const editingId = ref<string | null>(null)
const editingName = ref('')

function addHospital() {
  const name = newHospital.value.trim()
  if (!name) return
  if (store.state.hospitals.some((h) => h.name === name)) {
    window.alert('该医院已在字典中')
    return
  }
  store.addHospital(name)
  newHospital.value = ''
}

function startRename(id: string, current: string) {
  editingId.value = id
  editingName.value = current
}

function commitRename(id: string) {
  const name = editingName.value.trim()
  const hospital = store.state.hospitals.find((h) => h.id === id)
  if (hospital && name && name !== hospital.name) {
    if (store.state.hospitals.some((h) => h.id !== id && h.name === name)) {
      window.alert('该医院已在字典中')
    } else {
      store.updateHospital(id, { name })
    }
  }
  editingId.value = null
}

function removeHospital(id: string, name: string) {
  if (window.confirm(`确定从常用字典删除「${name}」吗？已有就医记录不受影响。`)) {
    store.deleteHospital(id)
  }
}

function addDepartment(hospitalId: string) {
  const dept = (deptInputs[hospitalId] ?? '').trim()
  if (!dept) return
  store.addDepartment(hospitalId, dept)
  deptInputs[hospitalId] = ''
}
</script>

<template>
  <div class="dict-manager">
    <div class="dict-add">
      <input
        v-model="newHospital"
        class="input"
        placeholder="添加常用医院，如：市第一人民医院"
        @keydown.enter="addHospital"
      />
      <button type="button" class="btn btn-primary" @click="addHospital">添加</button>
    </div>
    <p class="dict-tip">
      录入就医记录时，这里的医院和科室会自动补全；临时输入的未收录名称保存后会自动加入字典。
    </p>

    <div v-if="store.state.hospitals.length" class="dict-list">
      <div v-for="h in store.state.hospitals" :key="h.id" class="dict-item">
        <div class="dict-item-head">
          <input
            v-if="editingId === h.id"
            v-model="editingName"
            class="input input-sm dict-rename"
            @keydown.enter="commitRename(h.id)"
            @keydown.esc="editingId = null"
            @blur="commitRename(h.id)"
          />
          <strong v-else class="dict-name">{{ h.name }}</strong>
          <div class="dict-item-actions">
            <button type="button" class="btn btn-sm btn-ghost" @click="startRename(h.id, h.name)">
              改名
            </button>
            <button
              type="button"
              class="btn btn-sm btn-danger-ghost"
              @click="removeHospital(h.id, h.name)"
            >
              删除
            </button>
          </div>
        </div>
        <div class="dept-tags">
          <span v-for="d in h.departments" :key="d" class="dept-tag">
            {{ d }}
            <button
              type="button"
              class="dept-remove"
              :aria-label="`删除科室 ${d}`"
              @click="store.removeDepartment(h.id, d)"
            >
              ×
            </button>
          </span>
          <input
            v-model="deptInputs[h.id]"
            class="input input-sm dept-input"
            placeholder="＋ 科室，回车添加"
            @keydown.enter="addDepartment(h.id)"
          />
        </div>
      </div>
    </div>
    <EmptyState v-else icon="🏥" text="暂无常用医院，添加后录入就医记录时可自动补全" />
  </div>
</template>

<style scoped>
.dict-add {
  display: flex;
  gap: 10px;
}
.dict-add .input {
  flex: 1;
}
.dict-tip {
  margin: 10px 0 16px;
  font-size: 12px;
  color: var(--text-secondary);
}
.dict-list {
  display: flex;
  flex-direction: column;
  gap: 12px;
}
.dict-item {
  border: 1px solid var(--border-color);
  border-radius: 10px;
  padding: 12px 14px;
}
.dict-item-head {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 10px;
}
.dict-name {
  font-size: 15px;
}
.dict-rename {
  flex: 1;
}
.dict-item-actions {
  display: flex;
  gap: 8px;
  flex-shrink: 0;
}
.dept-tags {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 8px;
  margin-top: 10px;
}
.dept-tag {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  padding: 3px 8px;
  background: var(--accent-bg);
  color: var(--primary-color);
  border-radius: 999px;
  font-size: 13px;
}
.dept-remove {
  border: none;
  background: transparent;
  color: var(--text-secondary);
  cursor: pointer;
  font-size: 14px;
  line-height: 1;
  padding: 0;
}
.dept-remove:hover {
  color: var(--danger-color);
}
.dept-input {
  width: 150px;
}
.input-sm {
  padding: 5px 10px;
  font-size: 13px;
}
</style>
