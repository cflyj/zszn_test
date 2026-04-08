<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'

const activeTab = ref('stats')
const typeFilterOpen = ref(false)
const typeSearch = ref('')
const typeDropdownRef = ref(null)
const typeTriggerRef = ref(null)

const typeOptions = ['AAAA', 'BBBB', 'CCCC', 'DDDD']
const checkedTypes = ref([])

const rows = [
  { name: '整数001', type: 'AAAA', contact: '10887388928' },
  { name: '整数002', type: 'BBBB', contact: '10887388928' },
  { name: '整数003', type: 'CCCC', contact: '10887388928' },
  { name: '整数004', type: 'DDDD', contact: '10887388928' },
]

const filteredTypeOptions = computed(() => {
  const q = typeSearch.value.trim().toLowerCase()
  if (!q) return typeOptions
  return typeOptions.filter((t) => t.toLowerCase().includes(q))
})

const filteredRows = computed(() => {
  if (checkedTypes.value.length === 0) return rows
  return rows.filter((r) => checkedTypes.value.includes(r.type))
})

function toggleTypeFilter() {
  typeFilterOpen.value = !typeFilterOpen.value
}

function isTypeChecked(t) {
  return checkedTypes.value.includes(t)
}

function toggleTypeOption(t) {
  const i = checkedTypes.value.indexOf(t)
  if (i === -1) checkedTypes.value = [...checkedTypes.value, t]
  else checkedTypes.value = checkedTypes.value.filter((x) => x !== t)
}

function onDocClick(e) {
  if (!typeFilterOpen.value) return
  const el = typeDropdownRef.value
  const tr = typeTriggerRef.value
  if (el && !el.contains(e.target) && tr && !tr.contains(e.target)) {
    typeFilterOpen.value = false
  }
}

onMounted(() => document.addEventListener('click', onDocClick))
onUnmounted(() => document.removeEventListener('click', onDocClick))
</script>

<template>
  <div class="shell">
    <div class="panel" role="region" aria-label="订单表格">
      <h1 class="panel__title">Title</h1>
      <div class="panel__rule" />

      <div class="tabs" role="tablist">
        <button
          type="button"
          role="tab"
          :aria-selected="activeTab === 'stats'"
          class="tabs__btn"
          :class="{ 'tabs__btn--active': activeTab === 'stats' }"
          @click="activeTab = 'stats'"
        >
          订单统计
        </button>
        <button
          type="button"
          role="tab"
          :aria-selected="activeTab === 'list'"
          class="tabs__btn"
          :class="{ 'tabs__btn--active': activeTab === 'list' }"
          @click="activeTab = 'list'"
        >
          订单列表
        </button>
      </div>

      <div class="table-wrap">
        <table class="grid">
          <thead>
            <tr>
              <th class="grid__th grid__th--left">名称</th>
              <th class="grid__th grid__th--center">
                <span class="grid__th-inner">
                  类型
                  <button
                    ref="typeTriggerRef"
                    type="button"
                    class="icon-btn"
                    aria-label="按类型筛选"
                    :aria-expanded="typeFilterOpen"
                    @click.stop="toggleTypeFilter"
                  >
                    <img
                      class="icon-img icon-img--filter"
                      src="/icon_filter@2x.png"
                      width="14"
                      height="14"
                      alt=""
                      draggable="false"
                    />
                  </button>
                </span>
                <div
                  v-show="typeFilterOpen"
                  ref="typeDropdownRef"
                  class="dropdown"
                  role="dialog"
                  aria-label="类型筛选"
                  @click.stop
                >
                  <div class="dropdown__search">
                    <img
                      class="icon-img icon-img--search"
                      src="/icon_search@2x.png"
                      width="14"
                      height="14"
                      alt=""
                      draggable="false"
                    />
                    <input
                      v-model="typeSearch"
                      type="search"
                      class="dropdown__input"
                      placeholder="搜索"
                      aria-label="搜索类型"
                    />
                  </div>
                  <ul class="dropdown__list">
                    <li v-for="t in filteredTypeOptions" :key="t" class="dropdown__item">
                      <label class="check">
                        <input
                          type="checkbox"
                          class="check__input"
                          :checked="isTypeChecked(t)"
                          @change="toggleTypeOption(t)"
                        />
                        <span class="check__box" aria-hidden="true" />
                        <span class="check__label">{{ t }}</span>
                      </label>
                    </li>
                  </ul>
                </div>
              </th>
              <th class="grid__th grid__th--center">
                <span class="grid__th-inner">
                  联系方式
                  <span class="icon-btn icon-btn--static" aria-hidden="true">
                    <img
                      class="icon-img icon-img--filter"
                      src="/icon_filter@2x.png"
                      width="14"
                      height="14"
                      alt=""
                      draggable="false"
                    />
                  </span>
                </span>
              </th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(row, i) in filteredRows" :key="i">
              <td class="grid__td grid__td--left">{{ row.name }}</td>
              <td class="grid__td grid__td--center">{{ row.type }}</td>
              <td class="grid__td grid__td--center">{{ row.contact }}</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<style scoped>
.shell {
  box-sizing: border-box;
  width: 736px;
  height: 407px;
  padding: 20px 24px 16px;
  background: #000000;
  color: #ffffff;
  font-family: system-ui, 'Segoe UI', Roboto, 'PingFang SC', sans-serif;
  font-size: 13px;
  line-height: 1.35;
}

.panel {
  display: flex;
  flex-direction: column;
  height: 100%;
  min-height: 0;
}

.panel__title {
  margin: 0;
  font-size: 20px;
  font-weight: 600;
  letter-spacing: 0.02em;
  color: #ffffff;
  line-height: 1.2;
}

.panel__rule {
  height: 1px;
  background: #3a3a3a;
  margin-top: 12px;
  margin-bottom: 10px;
}

.tabs {
  display: flex;
  gap: 28px;
  align-items: flex-end;
  border-bottom: 1px solid transparent;
  margin-bottom: 12px;
}

.tabs__btn {
  padding: 0 0 8px;
  margin: 0;
  border: none;
  background: none;
  cursor: pointer;
  font: inherit;
  color: #8a8a8a;
  position: relative;
}

.tabs__btn--active {
  color: #ffffff;
}

.tabs__btn--active::after {
  content: '';
  position: absolute;
  left: 0;
  right: 0;
  bottom: 0;
  height: 2px;
  background: #ffffff;
}

.table-wrap {
  flex: 1;
  min-height: 0;
  border: 1px solid #3a3a3a;
  border-radius: 10px;
  overflow: hidden;
  display: flex;
  flex-direction: column;
}

.grid {
  width: 100%;
  border-collapse: collapse;
  table-layout: fixed;
}

.grid__th,
.grid__td {
  border: 1px solid #3a3a3a;
  padding: 10px 14px;
  font-weight: 400;
}

.grid__th {
  background: #000000;
  color: #ffffff;
  font-weight: 500;
  vertical-align: middle;
  position: relative;
}

.grid__th--left,
.grid__td--left {
  text-align: left;
  width: 34%;
}

.grid__th--center,
.grid__td--center {
  text-align: center;
  width: 33%;
}

.grid__th-inner {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
}

.icon-btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  padding: 2px;
  border: none;
  background: transparent;
  color: #c4c4c4;
  cursor: pointer;
  border-radius: 4px;
}

.icon-btn:hover,
.icon-btn:focus-visible {
  color: #ffffff;
}

.icon-btn:focus-visible {
  outline: 1px solid #ffffff;
  outline-offset: 2px;
}

.icon-btn--static {
  cursor: default;
  pointer-events: none;
}

.icon-img {
  display: block;
  flex-shrink: 0;
  object-fit: contain;
  pointer-events: none;
  user-select: none;
}

.icon-img--filter {
  width: 14px;
  height: 14px;
}

.icon-img--search {
  width: 14px;
  height: 14px;
  opacity: 0.85;
}

.dropdown {
  position: absolute;
  top: calc(100% + 6px);
  left: 50%;
  transform: translateX(-50%);
  z-index: 10;
  min-width: 168px;
  padding: 10px 10px 8px;
  background: #1a1a1a;
  border: 1px solid #3a3a3a;
  border-radius: 8px;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.45);
}

.dropdown__search {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 6px 8px;
  border: 1px solid #3a3a3a;
  border-radius: 6px;
  margin-bottom: 8px;
  background: #0f0f0f;
}

.dropdown__input {
  flex: 1;
  min-width: 0;
  border: none;
  background: transparent;
  color: #ffffff;
  font: inherit;
  outline: none;
}

.dropdown__input::placeholder {
  color: #6b6b6b;
}

.dropdown__list {
  list-style: none;
  margin: 0;
  padding: 0;
}

.dropdown__item {
  padding: 4px 0;
}

.check {
  display: flex;
  align-items: center;
  gap: 8px;
  cursor: pointer;
  color: #e8e8e8;
  user-select: none;
}

.check__input {
  position: absolute;
  opacity: 0;
  width: 0;
  height: 0;
}

.check__box {
  width: 14px;
  height: 14px;
  border: 1px solid #6b6b6b;
  border-radius: 2px;
  flex-shrink: 0;
  background: #000;
}

.check__input:focus-visible + .check__box {
  outline: 1px solid #ffffff;
  outline-offset: 2px;
}

.check__input:checked + .check__box {
  background: #ffffff;
  border-color: #ffffff;
}

.check__label {
  font-size: 13px;
}
</style>
