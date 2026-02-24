<template>
  <div class="cm-root">
    <!-- ══════════════ 顶部导航栏 ══════════════ -->
    <header class="cm-topbar">
      <div class="cm-topbar-inner">
        <div class="cm-brand">
          <span class="cm-brand-icon">
            <svg width="22" height="22" viewBox="0 0 24 24" fill="none">
              <rect x="3"  y="3"  width="8" height="8" rx="1.5" fill="#2563EB"/>
              <rect x="13" y="3"  width="8" height="8" rx="1.5" fill="#60A5FA"/>
              <rect x="3"  y="13" width="8" height="8" rx="1.5" fill="#93C5FD"/>
              <rect x="13" y="13" width="8" height="8" rx="1.5" fill="#BFDBFE"/>
            </svg>
          </span>
          <span class="cm-brand-name">客户档案管理系统</span>
        </div>
        <div class="cm-topbar-right">
          <span class="cm-date-chip">{{ currentDate }}</span>
          <div class="cm-avatar">五</div>
        </div>
      </div>
    </header>

    <!-- ══════════════ 主体 ══════════════ -->
    <main class="cm-main">

      <!-- KPI 卡片 -->
      <section class="cm-kpi-row">
        <div class="cm-kpi-card" v-for="kpi in kpiCards" :key="kpi.key"
             :class="`cm-kpi--${kpi.key}`">
          <div class="cm-kpi-left">
            <p class="cm-kpi-label">{{ kpi.label }}</p>
            <p class="cm-kpi-value">{{ kpi.value }}</p>
          </div>
          <div class="cm-kpi-icon-wrap">
            <svg v-html="kpi.svg" width="28" height="28" viewBox="0 0 24 24" fill="none"></svg>
          </div>
        </div>
      </section>

      <!-- 筛选面板 -->
      <section class="cm-filter-card">
        <div class="cm-filter-title">
          <svg width="15" height="15" viewBox="0 0 24 24" fill="none">
            <path d="M3 6h18M7 12h10M11 18h2" stroke="#2563EB" stroke-width="2" stroke-linecap="round"/>
          </svg>
          筛选条件
        </div>
        <div class="cm-filter-grid">
          <div class="cm-filter-item cm-filter-item--wide">
            <label class="cm-filter-label">关键词搜索</label>
            <div class="cm-input-wrap">
              <svg class="cm-input-icon" width="14" height="14" viewBox="0 0 24 24" fill="none">
                <circle cx="11" cy="11" r="8" stroke="#94A3B8" stroke-width="2"/>
                <path d="M21 21l-4.35-4.35" stroke="#94A3B8" stroke-width="2" stroke-linecap="round"/>
              </svg>
              <input
                  v-model="filters.keyword"
                  type="text"
                  placeholder="搜索客户全称 / 客户经理 / 部门 / 网格等"
                  class="cm-native-input cm-native-input--search"
                  @input="handleSearchInput"
              />
              <button v-if="filters.keyword" class="cm-input-clear" @click="filters.keyword = ''; fetchData()">✕</button>
            </div>
          </div>

          <div class="cm-filter-item">
            <label class="cm-filter-label">部门</label>
            <select v-model="filters.dept" class="cm-native-select" @change="fetchData">
              <option value="">全部部门</option>
              <option v-for="d in deptOptions" :key="d" :value="d">{{ d }}</option>
            </select>
          </div>

          <div class="cm-filter-item">
            <label class="cm-filter-label">网格</label>
            <select v-model="filters.grid" class="cm-native-select" @change="fetchData">
              <option value="">全部网格</option>
              <option v-for="g in gridOptions" :key="g" :value="g">{{ g }}</option>
            </select>
          </div>

          <div class="cm-filter-item">
            <label class="cm-filter-label">客户经理</label>
            <select v-model="filters.manager" class="cm-native-select" @change="fetchData">
              <option value="">全部经理</option>
              <option v-for="m in managerOptions" :key="m" :value="m">{{ m }}</option>
            </select>
          </div>

          <div class="cm-filter-item">
            <label class="cm-filter-label">点检情况</label>
            <select v-model="filters.status" class="cm-native-select" @change="fetchData">
              <option value="">全部状态</option>
              <option value="passed">已通过</option>
              <option value="pending">待审核</option>
              <option value="failed">未通过</option>
            </select>
          </div>

          <div class="cm-filter-item">
            <label class="cm-filter-label">一户一案</label>
            <select v-model="filters.hasFile" class="cm-native-select" @change="fetchData">
              <option value="">全部</option>
              <option value="yes">已上传</option>
              <option value="no">未上传</option>
            </select>
          </div>

          <div class="cm-filter-item">
            <label class="cm-filter-label">上传时间起</label>
            <input type="date" v-model="filters.dateStart" class="cm-native-input" @change="fetchData" />
          </div>

          <div class="cm-filter-item">
            <label class="cm-filter-label">上传时间止</label>
            <input type="date" v-model="filters.dateEnd" class="cm-native-input" @change="fetchData" />
          </div>

          <div class="cm-filter-item cm-filter-item--actions">
            <button class="cm-btn cm-btn--primary" @click="fetchData">查询</button>
            <button class="cm-btn cm-btn--ghost"   @click="resetFilter">重置</button>
          </div>
        </div>
      </section>

      <!-- 表格区域 -->
      <section class="cm-table-card">
        <div class="cm-table-toolbar">
          <span class="cm-table-count">共 <strong>{{ total }}</strong> 条记录</span>
          <button class="cm-btn cm-btn--sm cm-btn--ghost" @click="fetchData">↺ 刷新</button>
        </div>

        <div class="cm-table-wrap" :class="{ 'cm-table-wrap--loading': loading }">
          <div v-if="loading" class="cm-loading-mask">
            <div class="cm-spinner"></div>
            <span>加载中…</span>
          </div>

          <table class="cm-table">
            <thead>
            <tr>
              <th class="cm-th cm-th--index">序号</th>
              <th class="cm-th">部门</th>
              <th class="cm-th">网格</th>
              <th class="cm-th">客户经理</th>
              <th class="cm-th cm-th--wide">客户全称</th>
              <th class="cm-th">上传时间</th>
              <th class="cm-th cm-th--file">一户一案</th>
              <th class="cm-th">点检情况</th>
              <th class="cm-th cm-th--action">操作</th>
            </tr>
            </thead>
            <tbody>
            <tr v-if="!loading && tableData.length === 0">
              <td colspan="9" class="cm-empty">
                <svg width="48" height="48" viewBox="0 0 24 24" fill="none" style="opacity:.25;margin-bottom:8px">
                  <path d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2" stroke="#94A3B8" stroke-width="1.5" stroke-linecap="round"/>
                </svg>
                <p>暂无数据</p>
              </td>
            </tr>

            <tr v-for="(row, idx) in tableData" :key="row.id"
                class="cm-tr" :class="{ 'cm-tr--odd': idx % 2 !== 0 }">

              <td class="cm-td cm-td--center cm-td--index">
                {{ (currentPage - 1) * pageSize + idx + 1 }}
              </td>

              <td class="cm-td">
                <span class="cm-tag cm-tag--blue">{{ row.dept }}</span>
              </td>

              <td class="cm-td">
                <span class="cm-tag cm-tag--teal">{{ row.grid }}</span>
              </td>

              <td class="cm-td">
                <div class="cm-manager-cell">
                  <span class="cm-avatar-sm">{{ row.manager?.charAt(0) }}</span>
                  {{ row.manager }}
                </div>
              </td>

              <td class="cm-td cm-td--customer">{{ row.customerName }}</td>

              <td class="cm-td cm-td--center cm-td--time">{{ row.uploadTime || '—' }}</td>

              <!-- ══ 一户一案 — 上传 + 下载链接 ══ -->
              <td class="cm-td cm-td--file">
                <div class="cm-file-cell">
                  <!-- 已有文件：显示可点击下载的链接 -->
                  <a
                      v-if="row.caseFile"
                      :href="row.caseFile.url"
                      :download="row.caseFile.name"
                      class="cm-file-link"
                      target="_blank"
                      :title="row.caseFile.name"
                  >
                    <svg width="13" height="13" viewBox="0 0 24 24" fill="none" style="flex-shrink:0">
                      <path d="M14 2H6a2 2 0 00-2 2v16a2 2 0 002 2h12a2 2 0 002-2V8l-6-6z" stroke="#2563EB" stroke-width="1.8" stroke-linejoin="round"/>
                      <path d="M14 2v6h6M12 12v6M9 15l3 3 3-3" stroke="#2563EB" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"/>
                    </svg>
                    <span class="cm-file-name">{{ row.caseFile.name }}</span>
                  </a>
                  <span v-else class="cm-no-file">未上传</span>

                  <!-- 隐藏的原生 file input，每行独立 -->
                  <input
                      :ref="el => { if (el) fileInputRefs[row.id] = el }"
                      type="file"
                      accept=".xlsx,.xls"
                      style="display:none"
                      @change="e => handleFileChange(e, row)"
                  />

                  <!-- 上传按钮：点击触发对应行的 input -->
                  <button
                      class="cm-btn cm-btn--xs"
                      :class="row.caseFile ? 'cm-btn--amber' : 'cm-btn--primary'"
                      :disabled="uploadingId === row.id"
                      @click="triggerFileInput(row.id)"
                  >
                    <template v-if="uploadingId === row.id">
                      <span class="cm-btn-spinner"></span>
                      上传中…
                    </template>
                    <template v-else>
                      <svg width="11" height="11" viewBox="0 0 24 24" fill="none">
                        <path d="M21 15v4a2 2 0 01-2 2H5a2 2 0 01-2-2v-4M17 8l-5-5-5 5M12 3v12" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
                      </svg>
                      {{ row.caseFile ? '重新上传' : '上传文件' }}
                    </template>
                  </button>
                </div>
              </td>

              <td class="cm-td cm-td--center">
                  <span :class="['cm-badge', `cm-badge--${row.checkStatus}`]">
                    {{ statusLabel(row.checkStatus) }}
                  </span>
              </td>

              <td class="cm-td cm-td--center">
                <button class="cm-link-btn cm-link-btn--blue" @click="viewDetail(row)">详情</button>
                <span class="cm-op-divider">|</span>
                <button class="cm-link-btn cm-link-btn--amber" @click="editRow(row)">编辑</button>
              </td>

            </tr>
            </tbody>
          </table>
        </div>

        <!-- 分页 -->
        <div class="cm-pager-wrap">
          <span class="cm-pager-info">第 {{ currentPage }} 页 / 共 {{ totalPages }} 页</span>
          <div class="cm-pager-controls">
            <button class="cm-pager-btn" :disabled="currentPage <= 1" @click="onPageChange(currentPage - 1)">‹</button>
            <template v-for="p in pageNumbers" :key="p">
              <button v-if="p !== '...'"
                      class="cm-pager-btn"
                      :class="{ 'cm-pager-btn--active': p === currentPage }"
                      @click="onPageChange(p)">{{ p }}</button>
              <span v-else class="cm-pager-ellipsis">…</span>
            </template>
            <button class="cm-pager-btn" :disabled="currentPage >= totalPages" @click="onPageChange(currentPage + 1)">›</button>
          </div>
          <div class="cm-pager-size">
            每页
            <select v-model="pageSize" class="cm-native-select cm-native-select--sm"
                    @change="onSizeChange(Number(pageSize))">
              <option :value="10">10</option>
              <option :value="20">20</option>
              <option :value="50">50</option>
            </select>
            条
          </div>
        </div>
      </section>
    </main>

    <!-- 详情弹窗 -->
    <CustomerDetailModal
        :visible="detailVisible"
        :row="detailRow"
        @close="detailVisible = false"
        @edit="handleEditFromModal"
    />

    <!-- 上传结果 Toast -->
    <Transition name="cm-toast-fade">
      <div v-if="toast.show" :class="['cm-toast', `cm-toast--${toast.type}`]">
        <svg v-if="toast.type === 'success'" width="16" height="16" viewBox="0 0 24 24" fill="none">
          <path d="M22 11.08V12a10 10 0 11-5.93-9.14M22 4L12 14.01l-3-3" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
        </svg>
        <svg v-else width="16" height="16" viewBox="0 0 24 24" fill="none">
          <circle cx="12" cy="12" r="10" stroke="currentColor" stroke-width="2"/>
          <path d="M12 8v4M12 16h.01" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
        </svg>
        {{ toast.msg }}
      </div>
    </Transition>
  </div>
</template>

<script setup>
import { ref, reactive, computed, onMounted, onBeforeUnmount } from 'vue'
import CustomerDetailModal from '@/components/CustomerDetailModal.vue'

// ── 配置 ──
const API_BASE      = 'http://localhost:3000'
const uploadHeaders = {}   // 如需 Token: { Authorization: 'Bearer xxx' }

// ── 日期 ──
const currentDate = ref('')
let dateTimer = null
function updateDate() {
  currentDate.value = new Date().toLocaleDateString('zh-CN', {
    year: 'numeric', month: '2-digit', day: '2-digit', weekday: 'short',
  })
}

// ── 筛选 ──
const filters = reactive({
  keyword: '', dept: '', grid: '', manager: '',
  status: '', hasFile: '', dateStart: '', dateEnd: '',
})

// ── 分页 ──
const currentPage = ref(1)
const pageSize    = ref(10)
const total       = ref(0)
const totalPages  = computed(() => Math.ceil(total.value / pageSize.value) || 1)

// ── 枚举选项 ──
const deptOptions    = ref([])
const gridOptions    = ref([])
const managerOptions = ref([])

// ── 统计 ──
const statusCount = reactive({ passed: 0, pending: 0, failed: 0 })

// ── 表格 ──
const tableData = ref([])
const loading   = ref(false)

// ── 上传：每行独立的 file input ref ──
const uploadingId  = ref(null)
const fileInputRefs = reactive({})   // { [rowId]: HTMLInputElement }

// ── 详情弹窗 ──
const detailVisible = ref(false)
const detailRow     = ref({})

// ── Toast ──
const toast = reactive({ show: false, type: 'success', msg: '' })
let toastTimer = null
function showToast(type, msg) {
  clearTimeout(toastTimer)
  Object.assign(toast, { show: true, type, msg })
  toastTimer = setTimeout(() => { toast.show = false }, 3000)
}

// ── 防抖 ──
let searchTimer = null

// ── KPI ──
const kpiCards = computed(() => [
  {
    key: 'total', label: '总记录数', value: total.value,
    svg: '<path d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2" stroke="#2563EB" stroke-width="1.8" stroke-linecap="round"/>',
  },
  {
    key: 'passed', label: '已通过', value: statusCount.passed,
    svg: '<path d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z" stroke="#16A34A" stroke-width="1.8" stroke-linecap="round"/>',
  },
  {
    key: 'pending', label: '待审核', value: statusCount.pending,
    svg: '<path d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z" stroke="#D97706" stroke-width="1.8" stroke-linecap="round"/>',
  },
  {
    key: 'failed', label: '未通过', value: statusCount.failed,
    svg: '<path d="M10 14l2-2m0 0l2-2m-2 2l-2-2m2 2l2 2m7-2a9 9 0 11-18 0 9 9 0 0118 0z" stroke="#DC2626" stroke-width="1.8" stroke-linecap="round"/>',
  },
])

// ── 页码 ──
const pageNumbers = computed(() => {
  const tp = totalPages.value
  const cur = currentPage.value
  if (tp <= 7) return Array.from({ length: tp }, (_, i) => i + 1)
  const pages = [1]
  if (cur > 3) pages.push('...')
  for (let i = Math.max(2, cur - 1); i <= Math.min(tp - 1, cur + 1); i++) pages.push(i)
  if (cur < tp - 2) pages.push('...')
  pages.push(tp)
  return pages
})

// ──────────────────────────────────────────────
// 接口
// ──────────────────────────────────────────────
async function fetchOptions() {
  try {
    const res  = await fetch(`${API_BASE}/api/customer/options`)
    const json = await res.json()
    if (json.code === 200) {
      deptOptions.value    = json.data.depts    || []
      gridOptions.value    = json.data.grids    || []
      managerOptions.value = json.data.managers || []
    }
  } catch (e) { console.warn('枚举加载失败', e) }
}

async function fetchData() {
  loading.value = true
  try {
    const params = new URLSearchParams({
      keyword:   filters.keyword,
      dept:      filters.dept,
      grid:      filters.grid,
      manager:   filters.manager,
      status:    filters.status,
      hasFile:   filters.hasFile,
      dateStart: filters.dateStart,
      dateEnd:   filters.dateEnd,
      page:      currentPage.value,
      pageSize:  pageSize.value,
    })
    const res  = await fetch(`${API_BASE}/api/customer/list?${params}`)
    const json = await res.json()
    if (json.code === 200) {
      tableData.value = json.data.list
      total.value     = json.data.total
      calcStatus(json.data.list)
    }
  } catch (e) {
    console.error('请求失败', e)
  } finally {
    loading.value = false
  }
}

function calcStatus(list) {
  statusCount.passed  = list.filter(r => r.checkStatus === 'passed').length
  statusCount.pending = list.filter(r => r.checkStatus === 'pending').length
  statusCount.failed  = list.filter(r => r.checkStatus === 'failed').length
}

function handleSearchInput() {
  clearTimeout(searchTimer)
  searchTimer = setTimeout(() => { currentPage.value = 1; fetchData() }, 300)
}

function resetFilter() {
  Object.assign(filters, {
    keyword: '', dept: '', grid: '', manager: '',
    status: '', hasFile: '', dateStart: '', dateEnd: '',
  })
  currentPage.value = 1
  fetchData()
}

function onPageChange(p) { currentPage.value = p; fetchData() }
function onSizeChange(val) { pageSize.value = val; currentPage.value = 1; fetchData() }

// ──────────────────────────────────────────────
// 上传逻辑，file input + fetch FormData
// ──────────────────────────────────────────────

/** 点击按钮时，找到对应行的隐藏 input 并触发它 */
function triggerFileInput(rowId) {
  const input = fileInputRefs[rowId]
  if (input) {
    input.value = ''          // 清空，确保同一文件可以再次选择
    input.click()
  }
}

/** 用户选好文件后触发，执行校验 + 上传 */
async function handleFileChange(event, row) {
  const file = event.target.files?.[0]
  if (!file) return

  // 校验格式
  if (!/\.(xlsx|xls)$/i.test(file.name)) {
    showToast('error', '仅支持 .xlsx / .xls 格式文件')
    return
  }
  // // 校验大小（10 MB）
  // if (file.size / 1024 / 1024 > 10) {
  //   showToast('error', '文件大小不能超过 10 MB')
  //   return
  // }

  uploadingId.value = row.id

  try {
    const formData = new FormData()
    formData.append('file', file)

    // 发起上传请求
    const res = await fetch(`${API_BASE}/api/customer/${row.id}/upload`, {
      method: 'POST',
      headers: uploadHeaders,   // 注意：不要手动设置 Content-Type，浏览器会自动加 boundary
      body: formData,
    })

    if (!res.ok) throw new Error(`HTTP ${res.status}`)

    const json = await res.json()

    if (json.code === 200) {
      // 在 tableData 中局部更新这一行，不重新请求整页
      const idx = tableData.value.findIndex(r => r.id === row.id)
      if (idx !== -1) {
        tableData.value[idx] = {
          ...tableData.value[idx],
          caseFile:   json.data,   // { name: string, url: string }
          uploadTime: new Date().toLocaleString('zh-CN', { hour12: false })
              .replace(/\//g, '-').replace(',', ''),
        }
      }
      showToast('success', `「${json.data.name}」上传成功`)
    } else {
      showToast('error', json.msg || '上传失败，请稍后重试')
    }
  } catch (e) {
    console.error('上传异常', e)
    showToast('error', '上传请求失败，请检查网络或后端服务')
  } finally {
    uploadingId.value = null
  }
}

// ── 详情 ──
function viewDetail(row) {
  detailRow.value     = { ...row }
  detailVisible.value = true
}
function handleEditFromModal(row) {
  detailVisible.value = false
  editRow(row)
}
function editRow(row) {
  alert('编辑功能还没做')
}

// ── 工具 ──
function statusLabel(s) {
  return { passed: '已通过', pending: '待审核', failed: '未通过' }[s] || '—'
}

// ── 生命周期 ──
onMounted(() => {
  updateDate()
  dateTimer = setInterval(updateDate, 60000)
  Promise.all([fetchOptions(), fetchData()])
})
onBeforeUnmount(() => {
  clearInterval(dateTimer)
  clearTimeout(searchTimer)
  clearTimeout(toastTimer)
})
</script>

<style scoped>
.cm-root {
  --c-bg:        #F5F7FB;
  --c-card:      #FFFFFF;
  --c-border:    #E4E8F0;
  --c-primary:   #2563EB;
  --c-primary-h: #1D4ED8;
  --c-primary-s: #EFF6FF;
  --c-text:      #1E293B;
  --c-text-2:    #64748B;
  --c-text-3:    #94A3B8;
  --c-success:   #16A34A;
  --c-success-s: #DCFCE7;
  --c-warning:   #D97706;
  --c-warning-s: #FEF3C7;
  --c-danger:    #DC2626;
  --c-danger-s:  #FEE2E2;
  --c-teal:      #0D9488;
  --c-teal-s:    #CCFBF1;
  --radius:      8px;
  --shadow-sm:   0 1px 3px rgba(0,0,0,.06), 0 1px 2px rgba(0,0,0,.04);
  --shadow-md:   0 4px 12px rgba(0,0,0,.08);

  font-family: 'PingFang SC', 'Noto Sans SC', 'Microsoft YaHei', sans-serif;
  background: var(--c-bg);
  height: 100vh;
  width: 100vw;
  color: var(--c-text);
  font-size: 14px;
}

/* ── 顶部导航 ── */
.cm-topbar {
  position: sticky; top: 0; z-index: 100;
  background: var(--c-card);
  border-bottom: 1px solid var(--c-border);
  box-shadow: var(--shadow-sm);
}
.cm-topbar-inner {
  max-width: 1600px; margin: 0 auto; padding: 0 24px;
  height: 56px; display: flex; align-items: center; justify-content: space-between;
}
.cm-brand { display: flex; align-items: center; gap: 10px; }
.cm-brand-icon {
  width: 36px; height: 36px; background: var(--c-primary-s);
  border-radius: 8px; display: flex; align-items: center; justify-content: center;
}
.cm-brand-name { font-size: 17px; font-weight: 700; color: var(--c-primary); letter-spacing: 0.04em; }
.cm-topbar-right { display: flex; align-items: center; gap: 12px; }
.cm-date-chip {
  font-size: 12px; color: var(--c-text-2);
  background: var(--c-bg); border: 1px solid var(--c-border);
  border-radius: 20px; padding: 3px 12px;
}
.cm-avatar {
  width: 32px; height: 32px; border-radius: 50%;
  background: var(--c-primary); color: #fff;
  font-size: 13px; font-weight: 600;
  display: flex; align-items: center; justify-content: center;
}

/* ── 主体 ── */
.cm-main {
  max-width: 1600px; margin: 0 auto;
  padding: 24px 24px 40px;
  display: flex; flex-direction: column; gap: 16px;
}

/* ── KPI ── */
.cm-kpi-row { display: grid; grid-template-columns: repeat(4, 1fr); gap: 14px; }
.cm-kpi-card {
  background: var(--c-card); border: 1px solid var(--c-border);
  border-radius: var(--radius); padding: 18px 20px;
  display: flex; align-items: center; justify-content: space-between;
  box-shadow: var(--shadow-sm); transition: box-shadow 0.2s, transform 0.2s;
}
.cm-kpi-card:hover { box-shadow: var(--shadow-md); transform: translateY(-1px); }
.cm-kpi-label { font-size: 12px; color: var(--c-text-2); margin-bottom: 6px; }
.cm-kpi-value { font-size: 28px; font-weight: 700; line-height: 1; }
.cm-kpi--total   .cm-kpi-value { color: var(--c-primary); }
.cm-kpi--total   .cm-kpi-icon-wrap { background: var(--c-primary-s); border-radius: 10px; padding: 10px; }
.cm-kpi--passed  .cm-kpi-value { color: var(--c-success); }
.cm-kpi--passed  .cm-kpi-icon-wrap { background: var(--c-success-s); border-radius: 10px; padding: 10px; }
.cm-kpi--pending .cm-kpi-value { color: var(--c-warning); }
.cm-kpi--pending .cm-kpi-icon-wrap { background: var(--c-warning-s); border-radius: 10px; padding: 10px; }
.cm-kpi--failed  .cm-kpi-value { color: var(--c-danger); }
.cm-kpi--failed  .cm-kpi-icon-wrap { background: var(--c-danger-s); border-radius: 10px; padding: 10px; }

/* ── 筛选卡片 ── */
.cm-filter-card {
  background: var(--c-card); border: 1px solid var(--c-border);
  border-radius: var(--radius); padding: 18px 20px; box-shadow: var(--shadow-sm);
}
.cm-filter-title {
  display: flex; align-items: center; gap: 6px;
  font-size: 13px; font-weight: 600; color: var(--c-text);
  margin-bottom: 14px; padding-bottom: 10px; border-bottom: 1px solid var(--c-border);
}
.cm-filter-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 12px 16px; align-items: end;
}
.cm-filter-item--wide { grid-column: 1 / 3; }
.cm-filter-item--actions { display: flex; gap: 8px; align-items: flex-end; }
.cm-filter-label { display: block; font-size: 12px; color: var(--c-text-2); margin-bottom: 5px; font-weight: 500; }

/* ── 搜索框 ── */
.cm-input-wrap { position: relative; display: flex; align-items: center; }
.cm-input-icon { position: absolute; left: 10px; pointer-events: none; flex-shrink: 0; }
.cm-input-clear {
  position: absolute; right: 8px;
  background: none; border: none; cursor: pointer;
  color: var(--c-text-3); font-size: 12px; padding: 2px;
  border-radius: 50%; line-height: 1; transition: color 0.15s;
}
.cm-input-clear:hover { color: var(--c-text-2); }
.cm-native-input--search { padding-left: 32px; padding-right: 28px; width: 100%; }

/* ── 原生控件 ── */
.cm-native-select,
.cm-native-input {
  width: 100%; height: 36px;
  border: 1px solid var(--c-border); border-radius: var(--radius);
  background: var(--c-bg); color: var(--c-text);
  font-size: 13px; padding: 0 10px; outline: none;
  appearance: none; -webkit-appearance: none;
  cursor: pointer; transition: border-color 0.15s, box-shadow 0.15s;
  font-family: inherit;
}
.cm-native-select {
  background-image: url("data:image/svg+xml,%3Csvg width='10' height='6' viewBox='0 0 10 6' xmlns='http://www.w3.org/2000/svg'%3E%3Cpath d='M1 1l4 4 4-4' stroke='%2394A3B8' stroke-width='1.5' stroke-linecap='round' fill='none'/%3E%3C/svg%3E");
  background-repeat: no-repeat; background-position: right 10px center; padding-right: 28px;
}
.cm-native-select:focus,
.cm-native-input:focus {
  border-color: var(--c-primary);
  box-shadow: 0 0 0 3px rgba(37,99,235,.10);
  background-color: #fff;
}
.cm-native-select--sm { height: 30px; font-size: 12px; }

/* ── 按钮 ── */
.cm-btn {
  display: inline-flex; align-items: center; gap: 5px;
  height: 36px; padding: 0 18px; border-radius: var(--radius);
  font-size: 13px; font-weight: 500; border: 1px solid transparent;
  cursor: pointer; transition: all 0.15s; white-space: nowrap; font-family: inherit;
}
.cm-btn--primary { background: var(--c-primary); color: #fff; border-color: var(--c-primary); }
.cm-btn--primary:hover:not(:disabled) { background: var(--c-primary-h); border-color: var(--c-primary-h); }
.cm-btn--ghost { background: transparent; color: var(--c-text-2); border-color: var(--c-border); }
.cm-btn--ghost:hover { background: var(--c-bg); color: var(--c-text); }
.cm-btn--amber { background: var(--c-warning-s); color: var(--c-warning); border-color: #FCD34D; }
.cm-btn--amber:hover:not(:disabled) { background: #FEF08A; }
.cm-btn--sm { height: 30px; padding: 0 12px; font-size: 12px; }
.cm-btn--xs { height: 26px; padding: 0 10px; font-size: 12px; border-radius: 5px; }
.cm-btn:disabled { opacity: 0.55; cursor: not-allowed; }

/* 按钮内 loading spinner */
.cm-btn-spinner {
  width: 11px; height: 11px;
  border: 2px solid currentColor;
  border-top-color: transparent;
  border-radius: 50%;
  animation: spin 0.65s linear infinite;
  flex-shrink: 0;
}

/* ── 表格卡片 ── */
.cm-table-card {
  background: var(--c-card); border: 1px solid var(--c-border);
  border-radius: var(--radius); box-shadow: var(--shadow-sm); overflow: hidden;
}
.cm-table-toolbar {
  display: flex; align-items: center; justify-content: space-between;
  padding: 14px 20px; border-bottom: 1px solid var(--c-border);
}
.cm-table-count { font-size: 13px; color: var(--c-text-2); }
.cm-table-count strong { color: var(--c-primary); font-size: 15px; }

/* ── Loading ── */
.cm-table-wrap { position: relative; overflow-x: auto; min-height: 120px; }
.cm-table-wrap--loading { pointer-events: none; }
.cm-loading-mask {
  position: absolute; inset: 0; z-index: 10;
  background: rgba(255,255,255,.78);
  display: flex; flex-direction: column;
  align-items: center; justify-content: center;
  gap: 10px; font-size: 13px; color: var(--c-text-2);
}
.cm-spinner {
  width: 28px; height: 28px;
  border: 3px solid var(--c-border);
  border-top-color: var(--c-primary);
  border-radius: 50%;
  animation: spin 0.7s linear infinite;
}
@keyframes spin { to { transform: rotate(360deg); } }

/* ── 表格本体 ── */
.cm-table { width: 100%; border-collapse: collapse; font-size: 13px; }
.cm-th {
  background: #F8FAFC; border-bottom: 2px solid var(--c-border);
  padding: 11px 14px; text-align: left;
  font-size: 12px; font-weight: 600; color: var(--c-text-2);
  letter-spacing: 0.04em; white-space: nowrap;
}
.cm-th--index  { width: 56px; text-align: center; }
.cm-th--wide   { min-width: 200px; }
.cm-th--file   { min-width: 200px; }
.cm-th--action { width: 110px; text-align: center; }

.cm-tr { transition: background 0.1s; }
.cm-tr--odd { background: #FAFBFD; }
.cm-tr:hover { background: var(--c-primary-s) !important; }

.cm-td {
  padding: 12px 14px; border-bottom: 1px solid var(--c-border);
  color: var(--c-text); vertical-align: middle;
}
.cm-td--center  { text-align: center; }
.cm-td--index   { text-align: center; color: var(--c-text-3); font-size: 12px; }
.cm-td--customer { font-weight: 500; }
.cm-td--time    { font-size: 12px; color: var(--c-text-2); white-space: nowrap; }
.cm-td--file    { }

.cm-empty {
  text-align: center; padding: 60px 20px;
  color: var(--c-text-3); font-size: 14px;
  display: flex; flex-direction: column; align-items: center; gap: 4px;
}

/* ── 标签 ── */
.cm-tag {
  display: inline-block; padding: 2px 8px; border-radius: 4px;
  font-size: 12px; font-weight: 500; white-space: nowrap;
}
.cm-tag--blue { background: var(--c-primary-s); color: var(--c-primary); }
.cm-tag--teal { background: var(--c-teal-s); color: var(--c-teal); }

/* ── 经理 ── */
.cm-manager-cell { display: flex; align-items: center; gap: 7px; }
.cm-avatar-sm {
  width: 26px; height: 26px; border-radius: 50%;
  background: linear-gradient(135deg, var(--c-primary), #60A5FA);
  color: #fff; font-size: 11px; font-weight: 600;
  display: flex; align-items: center; justify-content: center; flex-shrink: 0;
}

/* ── 文件单元格 ── */
.cm-file-cell {
  display: flex; flex-direction: column;
  align-items: flex-start; gap: 6px;
}

/* 已上传：下载链接 */
.cm-file-link {
  display: inline-flex; align-items: center; gap: 5px;
  color: var(--c-primary); font-size: 12px; font-weight: 500;
  text-decoration: none; max-width: 180px;
  transition: color 0.15s;
  border-bottom: 1px dashed var(--c-primary);
  padding-bottom: 1px;
}
.cm-file-link:hover { color: var(--c-primary-h); border-bottom-style: solid; }
.cm-file-name {
  overflow: hidden; text-overflow: ellipsis; white-space: nowrap;
  max-width: 140px; display: inline-block;
}

/* 未上传提示 */
.cm-no-file { font-size: 12px; color: var(--c-text-3); }

/* ── 状态徽章 ── */
.cm-badge {
  display: inline-flex; align-items: center; gap: 5px;
  padding: 3px 10px; border-radius: 20px;
  font-size: 12px; font-weight: 600; white-space: nowrap;
}
.cm-badge::before { content: ''; width: 6px; height: 6px; border-radius: 50%; flex-shrink: 0; }
.cm-badge--passed  { background: var(--c-success-s); color: var(--c-success); }
.cm-badge--passed::before  { background: var(--c-success); }
.cm-badge--pending { background: var(--c-warning-s); color: var(--c-warning); }
.cm-badge--pending::before { background: var(--c-warning); }
.cm-badge--failed  { background: var(--c-danger-s);  color: var(--c-danger); }
.cm-badge--failed::before  { background: var(--c-danger); }

/* ── 操作按钮 ── */
.cm-link-btn {
  background: none; border: none; font-size: 13px; cursor: pointer;
  padding: 2px 4px; border-radius: 4px; transition: background 0.15s, color 0.15s;
  font-family: inherit;
}
.cm-link-btn--blue  { color: var(--c-primary); }
.cm-link-btn--blue:hover  { background: var(--c-primary-s); }
.cm-link-btn--amber { color: var(--c-warning); }
.cm-link-btn--amber:hover { background: var(--c-warning-s); }
.cm-op-divider { color: var(--c-border); margin: 0 2px; }

/* ── 分页 ── */
.cm-pager-wrap {
  display: flex; align-items: center; justify-content: flex-end;
  gap: 12px; padding: 14px 20px; border-top: 1px solid var(--c-border); flex-wrap: wrap;
}
.cm-pager-info { font-size: 12px; color: var(--c-text-3); }
.cm-pager-controls { display: flex; gap: 4px; align-items: center; }
.cm-pager-btn {
  min-width: 32px; height: 32px; padding: 0 6px;
  border: 1px solid var(--c-border); border-radius: 6px;
  background: var(--c-card); color: var(--c-text-2); font-size: 13px;
  cursor: pointer; transition: all 0.15s;
  display: flex; align-items: center; justify-content: center; font-family: inherit;
}
.cm-pager-btn:hover:not(:disabled) { border-color: var(--c-primary); color: var(--c-primary); background: var(--c-primary-s); }
.cm-pager-btn--active { background: var(--c-primary) !important; color: #fff !important; border-color: var(--c-primary) !important; }
.cm-pager-btn:disabled { opacity: 0.4; cursor: not-allowed; }
.cm-pager-ellipsis { color: var(--c-text-3); font-size: 13px; padding: 0 2px; }
.cm-pager-size { display: flex; align-items: center; gap: 6px; font-size: 12px; color: var(--c-text-2); }
.cm-pager-size .cm-native-select { width: 68px; }

/* ── Toast 通知 ── */
.cm-toast {
  position: fixed; bottom: 28px; left: 50%; transform: translateX(-50%);
  z-index: 9999;
  display: inline-flex; align-items: center; gap: 8px;
  padding: 10px 20px; border-radius: 24px;
  font-size: 13px; font-weight: 500;
  box-shadow: 0 4px 20px rgba(0,0,0,.14);
  pointer-events: none;
}
.cm-toast--success { background: #ECFDF5; color: var(--c-success); border: 1px solid #A7F3D0; }
.cm-toast--error   { background: #FEF2F2; color: var(--c-danger);  border: 1px solid #FECACA; }

.cm-toast-fade-enter-active { transition: opacity 0.25s, transform 0.25s; }
.cm-toast-fade-leave-active { transition: opacity 0.2s, transform 0.2s; }
.cm-toast-fade-enter-from   { opacity: 0; transform: translateX(-50%) translateY(12px); }
.cm-toast-fade-leave-to     { opacity: 0; transform: translateX(-50%) translateY(8px); }
</style>