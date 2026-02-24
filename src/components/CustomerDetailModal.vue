<template>
  <Teleport to="body">
    <Transition name="cm-modal-fade">
      <div v-if="visible" class="cm-modal-overlay" @click.self="close">
        <Transition name="cm-modal-slide">
          <div v-if="visible" class="cm-modal-box" role="dialog" aria-modal="true">

            <!-- 头部 -->
            <div class="cm-modal-header">
              <div class="cm-modal-header-left">
                <span class="cm-modal-icon">
                  <svg width="18" height="18" viewBox="0 0 24 24" fill="none">
                    <path d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2" stroke="#2563EB" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"/>
                  </svg>
                </span>
                <span class="cm-modal-title">客户详情</span>
              </div>
              <button class="cm-modal-close" @click="close" aria-label="关闭">
                <svg width="16" height="16" viewBox="0 0 24 24" fill="none">
                  <path d="M18 6L6 18M6 6l12 12" stroke="#64748B" stroke-width="2" stroke-linecap="round"/>
                </svg>
              </button>
            </div>

            <!-- 客户名称 Banner -->
            <div class="cm-modal-banner">
              <div class="cm-modal-cust-name">{{ row.customerName }}</div>
              <span :class="['cm-badge', `cm-badge--${row.checkStatus}`]">
                {{ statusLabel(row.checkStatus) }}
              </span>
            </div>

            <!-- 详情字段网格 -->
            <div class="cm-modal-body">
              <div class="cm-modal-section-title">基本信息</div>
              <div class="cm-modal-grid">
                <div class="cm-modal-field" v-for="f in basicFields" :key="f.key">
                  <p class="cm-modal-field-label">{{ f.label }}</p>
                  <p class="cm-modal-field-value">{{ row[f.key] || '—' }}</p>
                </div>
              </div>

              <div class="cm-modal-divider"></div>
              <div class="cm-modal-section-title">档案信息</div>
              <div class="cm-modal-grid">
                <!-- 上传时间 -->
                <div class="cm-modal-field">
                  <p class="cm-modal-field-label">上传时间</p>
                  <p class="cm-modal-field-value">{{ row.uploadTime || '—' }}</p>
                </div>
                <!-- 一户一案 -->
                <div class="cm-modal-field">
                  <p class="cm-modal-field-label">一户一案文件</p>
                  <div class="cm-modal-field-value">
                    <a v-if="row.caseFile"
                       :href="row.caseFile.url"
                       :download="row.caseFile.name"
                       class="cm-file-link"
                       target="_blank">
                      <svg width="14" height="14" viewBox="0 0 24 24" fill="none" style="flex-shrink:0">
                        <path d="M14 2H6a2 2 0 00-2 2v16a2 2 0 002 2h12a2 2 0 002-2V8l-6-6z" stroke="#2563EB" stroke-width="1.8" stroke-linejoin="round"/>
                        <path d="M14 2v6h6M16 13H8M16 17H8M10 9H8" stroke="#2563EB" stroke-width="1.8" stroke-linecap="round"/>
                      </svg>
                      {{ row.caseFile.name }}
                    </a>
                    <span v-else class="cm-no-file">暂未上传</span>
                  </div>
                </div>
                <!-- 点检情况 -->
                <div class="cm-modal-field">
                  <p class="cm-modal-field-label">点检情况</p>
                  <div class="cm-modal-field-value">
                    <span :class="['cm-badge', `cm-badge--${row.checkStatus}`]">
                      {{ statusLabel(row.checkStatus) }}
                    </span>
                  </div>
                </div>
              </div>
            </div>

            <!-- 底部操作 -->
            <div class="cm-modal-footer">
              <button class="cm-btn cm-btn--ghost" @click="close">关闭</button>
              <button class="cm-btn cm-btn--primary" @click="$emit('edit', row)">
                <svg width="14" height="14" viewBox="0 0 24 24" fill="none">
                  <path d="M11 4H4a2 2 0 00-2 2v14a2 2 0 002 2h14a2 2 0 002-2v-7M18.5 2.5a2.121 2.121 0 013 3L12 15l-4 1 1-4 9.5-9.5z" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"/>
                </svg>
                编辑
              </button>
            </div>

          </div>
        </Transition>
      </div>
    </Transition>
  </Teleport>
</template>

<script setup>
defineProps({
  visible: { type: Boolean, default: false },
  row:     { type: Object,  default: () => ({}) },
})

const emit = defineEmits(['close', 'edit'])

function close() { emit('close') }

const basicFields = [
  { label: '部门',     key: 'dept' },
  { label: '网格',     key: 'grid' },
  { label: '客户经理', key: 'manager' },
]

function statusLabel(s) {
  return { passed: '已通过', pending: '待审核', failed: '未通过' }[s] || '—'
}
</script>

<style scoped>
/* ── 遮罩 ── */
.cm-modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(15, 23, 42, 0.45);
  backdrop-filter: blur(3px);
  z-index: 1000;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px;
}

/* ── 弹窗盒子 ── */
.cm-modal-box {
  width: 100%;
  max-width: 560px;
  background: #FFFFFF;
  border-radius: 12px;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.18), 0 4px 16px rgba(0, 0, 0, 0.08);
  overflow: hidden;
  display: flex;
  flex-direction: column;
}

/* ── 头部 ── */
.cm-modal-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 18px 22px;
  border-bottom: 1px solid #E4E8F0;
}
.cm-modal-header-left { display: flex; align-items: center; gap: 10px; }
.cm-modal-icon {
  width: 34px; height: 34px;
  background: #EFF6FF;
  border-radius: 8px;
  display: flex; align-items: center; justify-content: center;
}
.cm-modal-title { font-size: 16px; font-weight: 700; color: #1E293B; }
.cm-modal-close {
  width: 32px; height: 32px;
  border: 1px solid #E4E8F0;
  border-radius: 7px;
  background: #F8FAFC;
  cursor: pointer;
  display: flex; align-items: center; justify-content: center;
  transition: background 0.15s, border-color 0.15s;
}
.cm-modal-close:hover { background: #F1F5F9; border-color: #CBD5E1; }

/* ── Banner ── */
.cm-modal-banner {
  background: linear-gradient(135deg, #EFF6FF 0%, #F0F9FF 100%);
  border-bottom: 1px solid #DBEAFE;
  padding: 16px 22px;
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 12px;
}
.cm-modal-cust-name {
  font-size: 17px;
  font-weight: 700;
  color: #1E293B;
  line-height: 1.4;
  flex: 1;
  min-width: 0;
}

/* ── 正文 ── */
.cm-modal-body { padding: 20px 22px; overflow-y: auto; max-height: 55vh; }

.cm-modal-section-title {
  font-size: 11px;
  font-weight: 700;
  color: #94A3B8;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  margin-bottom: 12px;
}
.cm-modal-divider { border: none; border-top: 1px solid #E4E8F0; margin: 16px 0; }

.cm-modal-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 14px 20px;
}
.cm-modal-field {}
.cm-modal-field-label {
  font-size: 11px;
  color: #94A3B8;
  margin-bottom: 4px;
  font-weight: 500;
  letter-spacing: 0.04em;
}
.cm-modal-field-value {
  font-size: 14px;
  color: #1E293B;
  font-weight: 500;
  word-break: break-all;
  display: flex;
  align-items: center;
  gap: 4px;
}

/* ── 文件链接 ── */
.cm-file-link {
  display: inline-flex;
  align-items: center;
  gap: 5px;
  color: #2563EB;
  font-size: 13px;
  text-decoration: none;
  font-weight: 500;
  transition: color 0.15s;
  max-width: 100%;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
.cm-file-link:hover { color: #1D4ED8; text-decoration: underline; }
.cm-no-file { color: #94A3B8; font-size: 13px; }

/* ── 状态徽章 ── */
.cm-badge {
  display: inline-flex;
  align-items: center;
  gap: 5px;
  padding: 4px 11px;
  border-radius: 20px;
  font-size: 12px;
  font-weight: 600;
  white-space: nowrap;
  flex-shrink: 0;
}
.cm-badge::before { content: ''; width: 6px; height: 6px; border-radius: 50%; flex-shrink: 0; }
.cm-badge--passed  { background: #DCFCE7; color: #16A34A; }
.cm-badge--passed::before  { background: #16A34A; }
.cm-badge--pending { background: #FEF3C7; color: #D97706; }
.cm-badge--pending::before { background: #D97706; }
.cm-badge--failed  { background: #FEE2E2; color: #DC2626; }
.cm-badge--failed::before  { background: #DC2626; }

/* ── 底部 ── */
.cm-modal-footer {
  display: flex;
  align-items: center;
  justify-content: flex-end;
  gap: 10px;
  padding: 14px 22px;
  border-top: 1px solid #E4E8F0;
  background: #F8FAFC;
}

/* ── 按钮 ── */
.cm-btn {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  height: 36px;
  padding: 0 18px;
  border-radius: 8px;
  font-size: 13px;
  font-weight: 500;
  border: 1px solid transparent;
  cursor: pointer;
  transition: all 0.15s;
  white-space: nowrap;
}
.cm-btn--primary {
  background: #2563EB;
  color: #fff;
  border-color: #2563EB;
}
.cm-btn--primary:hover { background: #1D4ED8; border-color: #1D4ED8; }
.cm-btn--ghost {
  background: transparent;
  color: #64748B;
  border-color: #E4E8F0;
}
.cm-btn--ghost:hover { background: #F1F5F9; color: #1E293B; }

/* ── 过渡动画 ── */
.cm-modal-fade-enter-active,
.cm-modal-fade-leave-active { transition: opacity 0.22s ease; }
.cm-modal-fade-enter-from,
.cm-modal-fade-leave-to { opacity: 0; }

.cm-modal-slide-enter-active { transition: opacity 0.22s ease, transform 0.22s cubic-bezier(0.34, 1.56, 0.64, 1); }
.cm-modal-slide-leave-active { transition: opacity 0.18s ease, transform 0.18s ease; }
.cm-modal-slide-enter-from  { opacity: 0; transform: scale(0.94) translateY(12px); }
.cm-modal-slide-leave-to    { opacity: 0; transform: scale(0.96) translateY(6px); }
</style>
