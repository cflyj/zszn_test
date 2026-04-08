<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'

const activeTab = ref('stats')
const typeFilterOpen = ref(false)
const typeSearch = ref('')
const typeDropdownRef = ref(null)
const typeTriggerRef = ref(null)

const typeOptions = ['AAAA', 'BBBB', 'CCCC', 'DDDD']
/* 默认全选：表格有数据且与下拉勾选一致；全部取消勾选则表格不展示行 */
const checkedTypes = ref([...typeOptions])

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
  if (checkedTypes.value.length === 0) return []
  return rows.filter((r) => checkedTypes.value.includes(r.type))
})

/** 表体无数据时的说明（未勾选类型 / 已勾选但无匹配行） */
const emptyTableHint = computed(() => {
  if (checkedTypes.value.length === 0) {
    return '未选择类型：请点击「类型」旁的筛选图标，勾选需要展示的数据类型。'
  }
  return '暂无符合当前已选类型的数据，请调整筛选条件。'
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
      <aside class="shell__sidebar" aria-label="侧边栏" />
      <div class="shell__main" role="region" aria-label="订单表格">
        <div class="shell__title-box">
          <h1 class="shell__title">Title</h1>
        </div>

        <div class="shell__tabs tabs" role="tablist">
          <button
            type="button"
            role="tab"
            :aria-selected="activeTab === 'stats'"
            class="tabs__btn"
            :class="{ 'tabs__btn--active': activeTab === 'stats' }"
            @click="activeTab = 'stats'"
          >
            <span class="tabs__btn-label">订单统计</span>
          </button>
          <button
            type="button"
            role="tab"
            :aria-selected="activeTab === 'list'"
            class="tabs__btn"
            :class="{ 'tabs__btn--active': activeTab === 'list' }"
            @click="activeTab = 'list'"
          >
            <span class="tabs__btn-label">订单列表</span>
          </button>
        </div>

        <div class="shell__table">
          <div class="table-wrap">
        <table class="grid">
          <colgroup>
            <col class="grid__col grid__col--name" />
            <col class="grid__col grid__col--type" />
            <col class="grid__col grid__col--contact" />
          </colgroup>
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
                      width="16"
                      height="16"
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
        </table>
        <div class="table-wrap__body">
          <table class="grid">
            <colgroup>
              <col class="grid__col grid__col--name" />
              <col class="grid__col grid__col--type" />
              <col class="grid__col grid__col--contact" />
            </colgroup>
            <tbody>
              <template v-if="filteredRows.length === 0">
                <tr class="grid__empty-row">
                  <td
                    class="grid__td grid__empty"
                    colspan="3"
                    role="status"
                    aria-live="polite"
                  >
                    {{ emptyTableHint }}
                  </td>
                </tr>
              </template>
              <template v-else>
                <tr v-for="(row, i) in filteredRows" :key="i">
                  <td class="grid__td grid__td--left">{{ row.name }}</td>
                  <td class="grid__td grid__td--center">{{ row.type }}</td>
                  <td class="grid__td grid__td--center">{{ row.contact }}</td>
                </tr>
              </template>
            </tbody>
          </table>
          </div>
          </div>
        </div>
      </div>
    </div>
</template>

<style scoped>
.shell {
  isolation: isolate;
  box-sizing: border-box;
  width: 736px;
  height: 407px;
  padding: 0;
  position: relative;
  z-index: 1;
  display: flex;
  flex-direction: row;
  align-items: stretch;
  background: transparent;
  color: #ffffff;
  font-family: system-ui, 'Segoe UI', Roboto, 'PingFang SC', sans-serif;
  font-size: 13px;
  line-height: 1.35;
  /* 侧栏与表格同色；其余主区域为 #101010 @ 10% */
  --sidebar-table-bg: rgba(0, 0, 0);
  --main-bg: rgba(16, 16, 16, 0.1);
  --main-bg1: rgba(16, 16, 16);
  --grid-line: rgba(255, 255, 255, 0.14);
  /* 表头文字色（设计稿 var(--font, …)） */
  --font: rgba(150, 150, 150, 1);
}

.shell__sidebar {
  width: 80px;
  height: 407px;
  flex-shrink: 0;
  background: var(--sidebar-table-bg);
}

.shell__main {
  position: relative;
  flex: 1 1 656px;
  width: 656px;
  min-width: 0;
  height: 407px;
  box-sizing: border-box;
  background: var(--main-bg1);
}

/* Figma: 596×64, left 80 → 相对主区 left 0，贴侧栏 */
.shell__title-box {
  position: absolute;
  left: 0;
  top: 0;
  width: 596px;
  height: 64px;
  box-sizing: border-box;
  display: flex;
  align-items: center;
  padding: 0 20px;
}

/* 底边线左右各缩进 12px */
.shell__title-box::after {
  content: '';
  position: absolute;
  left: 12px;
  right: 12px;
  bottom: 0;
  height: 1px;
  background: rgba(255, 255, 255, 0.2);
}

.shell__title {
  margin: 0;
  font-family: 'PingFang SC', 'Microsoft YaHei', system-ui, sans-serif;
  font-weight: 600;
  font-size: 20px;
  line-height: 30px;
  letter-spacing: 1px;
  color: #ffffff;
}

/* 单项 83×40，gap 6 → 总宽约 172；top 80, left 104 → 主区内 left 24 */
.shell__tabs {
  position: absolute;
  left: 24px;
  top: 80px;
  width: 172px;
  height: 40px;
  box-sizing: border-box;
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: flex-start;
  gap: 6px;
  margin: 0;
  padding: 0;
}

.tabs__btn {
  box-sizing: border-box;
  width: 83px;
  height: 40px;
  margin: 0;
  padding: 0 12px;
  border: none;
  border-radius: 6px;
  background: transparent;
  cursor: pointer;
  color: #8a8a8a;
  flex: 0 0 83px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  text-align: center;
  font-family: 'PingFang SC', 'Microsoft YaHei', system-ui, sans-serif;
  font-weight: 400;
  font-size: 14px;
  line-height: 20px;
  letter-spacing: 1px;
  white-space: nowrap;
  transition: color 0.15s ease;
}

.tabs__btn-label {
  display: block;
  line-height: 20px;
}

/* 字体底部往下 5px、宽 70px 的下划线；未选中用透明条占位避免高度跳动 */
.tabs__btn::after {
  content: '';
  display: block;
  width: 70px;
  height: 2px;
  margin-top: 5px;
  flex-shrink: 0;
  background: transparent;
  border-radius: 1px;
}

.tabs__btn:hover {
  color: #e0e0e0;
}

.tabs__btn--active {
  color: #ffffff;
}

.tabs__btn--active::after {
  background: #ffffff;
}

.tabs__btn:focus-visible {
  outline: 1px solid rgba(255, 255, 255, 0.45);
  outline-offset: 2px;
}

/* Figma: 572×211, top 136, left 104 → 主区内 left 24 */
.shell__table {
  position: absolute;
  left: 24px;
  top: 136px;
  width: 572px;
  height: 211px;
  box-sizing: border-box;
  background: var(--sidebar-table-bg);
  border: 1px solid var(--grid-line);
  border-radius: 10px;
  overflow: visible;
  display: flex;
  flex-direction: column;
}

.shell__table .table-wrap {
  flex: 1 1 auto;
  min-height: 0;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  border: none;
  border-radius: 10px;
  overflow: visible;
  background: transparent;
}

.shell__table .table-wrap__body {
  flex: 1 1 auto;
  min-height: 0;
  max-height: none;
  overflow-y: auto;
  overflow-x: hidden;
  overscroll-behavior: contain;
  border-radius: 0 0 9px 9px;
}

.table-wrap__body .grid tbody tr:first-child td {
  border-top: none;
}

.grid {
  width: 570px;
  max-width: 100%;
  margin-left: auto;
  margin-right: auto;
  border-collapse: separate;
  border-spacing: 0;
  table-layout: fixed;
}

/* 列宽比例：120 + 210 + 240 = 570（与 572 容器边框内宽约一致） */
.grid__col--name {
  width: 120px;
}

.grid__col--type {
  width: 210px;
}

.grid__col--contact {
  width: 240px;
}

/* 仅右、下画线，避免相邻单元格双边框；外轮廓交给 .shell__table */
.grid__th,
.grid__td {
  border: 0 solid var(--grid-line);
  border-right-width: 1px;
  border-bottom-width: 1px;
  box-sizing: border-box;
  font-weight: 400;
}

.grid__th:last-child,
.grid__td:last-child {
  border-right: none;
}

.shell__table .table-wrap .grid thead .grid__th {
  border-top: none;
  border-bottom: 1px solid var(--grid-line);
}

.shell__table .table-wrap__body .grid tbody tr:last-child .grid__td {
  border-bottom: none;
}

.grid__th {
  height: 48px;
  padding: 0 12px;
  box-sizing: border-box;
  background: transparent;
  color: var(--font, rgba(150, 150, 150, 1));
  font-family: 'PingFang SC', 'Microsoft YaHei', system-ui, sans-serif;
  font-weight: 400;
  font-size: 14px;
  line-height: 16px;
  letter-spacing: 2px;
  text-align: center;
  vertical-align: middle;
  position: relative;
}

/* 「名称」表头左对齐，与下方第一列数据对齐（padding 与 .grid__th 一致） */
.grid__th--left {
  text-align: left;
  padding-left: 16px;
  padding-right: 12px;
}

.grid__td {
  height: 40px;
  padding: 0 12px;
  background: transparent;
  vertical-align: middle;
}

/* 表头：名称左偏，其余居中；数据行第一列左对齐 */
.grid__td--left {
  text-align: left;
}

.grid__td--center {
  text-align: center;
}

.grid__empty {
  height: auto;
  min-height: 96px;
  padding: 20px 28px;
  text-align: center;
  vertical-align: middle;
  color: rgba(255, 255, 255, 0.52);
  font-family: 'PingFang SC', 'Microsoft YaHei', system-ui, sans-serif;
  font-size: 13px;
  line-height: 1.55;
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
  z-index: 20;
  width: 216px;
  height: 186px;
  max-width: calc(100vw - 32px);
  box-sizing: border-box;
  padding: 8px;
  display: flex;
  flex-direction: column;
  align-items: center;
  background: rgba(0, 0, 0, 0.4);
  border: 1px solid rgba(67, 67, 67, 0.5);
  border-radius: 12px;
  backdrop-filter: blur(24px);
  overflow: hidden;
}

.dropdown__search {
  display: flex;
  align-items: center;
  flex-shrink: 0;
  box-sizing: border-box;
  width: 200px;
  height: 32px;
  padding: 0 10px 0 12px;
  gap: 8px;
  border: 0.5px solid rgba(67, 67, 67, 1);
  border-radius: 4px;
  background: transparent;
}

.dropdown .dropdown__search .icon-img--search {
  width: 16px;
  height: 16px;
  opacity: 1;
  flex-shrink: 0;
}

.dropdown__input {
  flex: 1;
  min-width: 0;
  height: 16px;
  line-height: 16px;
  border: none;
  background: transparent;
  color: #ffffff;
  outline: none;
  font-family: 'PingFang SC', 'Microsoft YaHei', system-ui, sans-serif;
  font-weight: 400;
  font-size: 12px;
  letter-spacing: 2px;
}

.dropdown__input::placeholder {
  color: rgba(255, 255, 255, 0.35);
  font-family: 'PingFang SC', 'Microsoft YaHei', system-ui, sans-serif;
  font-weight: 400;
  font-size: 12px;
  line-height: 16px;
  letter-spacing: 2px;
}

.dropdown__list {
  list-style: none;
  margin: 8px 0 0;
  padding: 0;
  width: 200px;
  flex: 1 1 auto;
  min-height: 0;
  overflow-y: auto;
  overscroll-behavior: contain;
}

.dropdown__item {
  box-sizing: border-box;
  width: 200px;
  height: 32px;
  flex-shrink: 0;
  padding: 0;
  margin: 0;
}

.check {
  display: flex;
  align-items: center;
  box-sizing: border-box;
  width: 100%;
  height: 100%;
  gap: 8px;
  padding: 0 8px;
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
  background: rgba(0, 0, 0, 0.4);
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
  font-family: 'PingFang SC', 'Microsoft YaHei', system-ui, sans-serif;
  font-size: 12px;
  line-height: 16px;
  letter-spacing: 1px;
}
</style>
