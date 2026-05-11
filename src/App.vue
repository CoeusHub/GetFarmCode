<script setup>
import { ref, onMounted, nextTick, watch } from 'vue'
import QRCode from 'qrcode'

const tab = ref('wx')
const wxCode = ref('')
const qqManualCode = ref('')
const wxQrcode = ref(null)
const copied = ref(false)

const WX_APPID = import.meta.env.VITE_WX_APPID
const REDIRECT_URI = encodeURIComponent(window.location.origin + window.location.pathname)
const qqAppQrUrl = import.meta.env.VITE_QQ_QR_URL

onMounted(() => {
  const params = new URLSearchParams(window.location.search)
  const code = params.get('code')
  if (code && params.get('state') === 'wx') {
    wxCode.value = code
    tab.value = 'wx'
    return
  }

  const ua = navigator.userAgent.toLowerCase()
  if (/micromessenger/i.test(ua) && WX_APPID !== 'wx你的测试号appid') {
    const authUrl = `https://open.weixin.qq.com/connect/oauth2/authorize?appid=${WX_APPID}&redirect_uri=${REDIRECT_URI}&response_type=code&scope=snsapi_userinfo&state=wx#wechat_redirect`
    window.location.href = authUrl
  }
})

watch(tab, async (val) => {
  if (val === 'wx' && !wxCode.value) {
    await nextTick()
    if (wxQrcode.value) {
      const authUrl = `https://open.weixin.qq.com/connect/oauth2/authorize?appid=${WX_APPID}&redirect_uri=${REDIRECT_URI}&response_type=code&scope=snsapi_userinfo&state=wx#wechat_redirect`
      QRCode.toCanvas(wxQrcode.value, authUrl, {
        width: 200,
        margin: 0,
        color: { dark: '#000000', light: '#ffffff' }
      })
    }
  }
}, { immediate: true })

function copy(text) {
  if (!text) return
  const done = () => {
    copied.value = true
    setTimeout(() => { copied.value = false }, 2000)
  }
  if (navigator.clipboard) {
    navigator.clipboard.writeText(text).then(done).catch(() => fallbackCopy(text, done))
  } else {
    fallbackCopy(text, done)
  }
}

function fallbackCopy(text, done) {
  const ta = document.createElement('textarea')
  ta.value = text
  ta.style.position = 'fixed'
  ta.style.left = '-9999px'
  document.body.appendChild(ta)
  ta.select()
  try { document.execCommand('copy'); done() } catch (e) { /* ignore */ }
  document.body.removeChild(ta)
}
</script>

<template>
  <div class="card">
    <div class="header">
      <div class="header-icon">🔑</div>
      <h1>获取登录 CODE</h1>
      <p>支持微信 &amp; QQ 双平台授权</p>
    </div>

    <div class="tabs">
      <button
        :class="{ active: tab === 'wx' }"
        @click="tab = 'wx'"
      >
        <span class="tab-dot wx"></span>微信登录
      </button>
      <button
        :class="{ active: tab === 'qq' }"
        @click="tab = 'qq'"
      >
        <span class="tab-dot qq"></span>QQ 登录
      </button>
    </div>

    <!-- ========== 微信 Tab ========== -->
    <div v-show="tab === 'wx'" class="tab-content">
      <template v-if="wxCode">
        <div class="success-banner">
          <span class="check">✓</span>
          <span>授权成功</span>
          <span class="platform-tag wx">微信</span>
        </div>
        <div class="code-terminal">
          <div class="terminal-bar">
            <span class="terminal-dot" style="background:#ff5f57"></span>
            <span class="terminal-dot" style="background:#febc2e"></span>
            <span class="terminal-dot" style="background:#28c840"></span>
            <span class="terminal-title">code</span>
          </div>
          <div class="code-content">{{ wxCode }}</div>
        </div>
        <button class="btn" :class="{ copied }" @click="copy(wxCode)">
          <span class="btn-text">📋 一键复制 CODE</span>
          <span class="btn-done">✅ 已复制到剪贴板</span>
        </button>
      </template>
      <template v-else>
        <div class="qrcode-wrapper">
          <div class="qrcode-frame">
            <canvas ref="wxQrcode" width="200" height="200"></canvas>
          </div>
        </div>
        <p class="hint">请使用 <strong>微信</strong> 扫描上方二维码</p>
      </template>
    </div>

    <!-- ========== QQ Tab ========== -->
    <div v-show="tab === 'qq'" class="tab-content">
      <p class="hint">请用 <strong>手机 QQ</strong> 扫描下方小程序预览码</p>
      <div class="qrcode-wrapper">
        <img
          :src="qqAppQrUrl"
          alt="QQ小程序预览码"
          class="qrcode-img"
        />
      </div>
      <p class="step-text">
        扫描后进入小程序 → 点击「获取 code」→ 复制 code
      </p>
      <textarea
        v-model="qqManualCode"
        placeholder="从小程序复制后粘贴到这里..."
        class="input-box"
        rows="3"
      ></textarea>
      <button
        class="btn"
        :class="{ copied: copied && qqManualCode }"
        :disabled="!qqManualCode"
        @click="copy(qqManualCode)"
      >
        <span class="btn-text">📋 复制此 Code</span>
        <span class="btn-done">✅ 已复制到剪贴板</span>
      </button>
    </div>

    <div class="footer-tip">code 有效期很短，请尽快使用</div>
  </div>
</template>

<style>
:root {
  --wechat: #07C160;
  --qq: #12B7F5;
  --bg: #08080f;
  --card: rgba(18, 18, 30, 0.75);
  --border: rgba(255, 255, 255, 0.06);
  --text: #e4e4ec;
  --muted: #7c7c8a;
  --code-bg: #0c0c16;
  --code-text: #00e676;
  --radius: 18px;
}

* { margin: 0; padding: 0; box-sizing: border-box; }

body {
  background: var(--bg);
  background-image:
    radial-gradient(ellipse 80% 60% at 50% -20%, rgba(7, 193, 96, 0.06), transparent),
    radial-gradient(ellipse 60% 50% at 80% 80%, rgba(18, 183, 245, 0.05), transparent);
  color: var(--text);
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "PingFang SC", "Microsoft YaHei", sans-serif;
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  padding: 20px;
  -webkit-font-smoothing: antialiased;
}

.card {
  width: 100%;
  max-width: 420px;
  background: var(--card);
  backdrop-filter: blur(24px);
  -webkit-backdrop-filter: blur(24px);
  border: 1px solid var(--border);
  border-radius: var(--radius);
  padding: 32px 24px 28px;
  box-shadow: 0 8px 40px rgba(0, 0, 0, 0.5), inset 0 1px 0 rgba(255, 255, 255, 0.04);
  animation: cardIn 0.5s ease-out;
}

@keyframes cardIn {
  from { opacity: 0; transform: translateY(16px) scale(0.98); }
  to   { opacity: 1; transform: translateY(0) scale(1); }
}

.header {
  text-align: center;
  margin-bottom: 20px;
}

.header-icon {
  width: 44px;
  height: 44px;
  background: linear-gradient(135deg, rgba(7, 193, 96, 0.2), rgba(18, 183, 245, 0.2));
  border-radius: 12px;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  font-size: 22px;
  margin-bottom: 12px;
}

.header h1 {
  font-size: 20px;
  font-weight: 600;
  letter-spacing: -0.3px;
}

.header p {
  font-size: 13px;
  color: var(--muted);
  margin-top: 4px;
}

.tabs {
  display: flex;
  justify-content: center;
  gap: 10px;
  margin-bottom: 24px;
}

.tabs button {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  padding: 9px 22px;
  border: 1.5px solid rgba(255, 255, 255, 0.1);
  background: rgba(255, 255, 255, 0.03);
  color: var(--muted);
  border-radius: 10px;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.25s;
  font-family: inherit;
}

.tabs button:hover {
  border-color: rgba(255, 255, 255, 0.2);
  color: var(--text);
}

.tabs button.active {
  background: rgba(255, 255, 255, 0.08);
  border-color: rgba(255, 255, 255, 0.15);
  color: #fff;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.3);
}

.tab-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  display: inline-block;
}

.tab-dot.wx { background: var(--wechat); }
.tab-dot.qq { background: var(--qq); }

.tab-content {
  animation: fadeUp 0.35s ease-out;
}

@keyframes fadeUp {
  from { opacity: 0; transform: translateY(8px); }
  to   { opacity: 1; transform: translateY(0); }
}

.qrcode-wrapper {
  display: flex;
  justify-content: center;
  margin-bottom: 16px;
}

.qrcode-frame {
  background: #fff;
  padding: 10px;
  border-radius: 14px;
  position: relative;
  box-shadow: 0 2px 16px rgba(0, 0, 0, 0.3);
  display: inline-block;
}

.qrcode-frame::after {
  content: "";
  position: absolute;
  inset: -2px;
  border-radius: 16px;
  padding: 2px;
  background: linear-gradient(135deg, var(--wechat), var(--qq));
  -webkit-mask: linear-gradient(#fff 0 0) content-box, linear-gradient(#fff 0 0);
  mask: linear-gradient(#fff 0 0) content-box, linear-gradient(#fff 0 0);
  -webkit-mask-composite: xor;
  mask-composite: exclude;
  opacity: 0.5;
}

.qrcode-img {
  width: 200px;
  height: 200px;
  border-radius: 12px;
  display: block;
  box-shadow: 0 2px 16px rgba(0, 0, 0, 0.3);
}

.hint {
  text-align: center;
  font-size: 14px;
  color: var(--muted);
  margin-bottom: 8px;
}

.hint strong { color: var(--text); }

.step-text {
  text-align: center;
  font-size: 13px;
  color: var(--muted);
  margin: 12px 0 16px;
  line-height: 1.6;
}

.success-banner {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 10px 14px;
  background: rgba(0, 200, 83, 0.1);
  border: 1px solid rgba(0, 200, 83, 0.2);
  border-radius: 10px;
  margin-bottom: 16px;
  font-size: 14px;
  font-weight: 500;
  color: #00c853;
}

.success-banner .check {
  width: 20px;
  height: 20px;
  background: #00c853;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 11px;
  color: #fff;
  flex-shrink: 0;
}

.platform-tag {
  font-size: 12px;
  padding: 2px 8px;
  border-radius: 4px;
  margin-left: auto;
}

.platform-tag.wx { background: rgba(7, 193, 96, 0.15); color: var(--wechat); }
.platform-tag.qq { background: rgba(18, 183, 245, 0.15); color: var(--qq); }

.code-terminal {
  background: var(--code-bg);
  border: 1px solid rgba(255, 255, 255, 0.06);
  border-radius: 10px;
  overflow: hidden;
  margin-bottom: 16px;
}

.terminal-bar {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 10px 14px;
  background: rgba(255, 255, 255, 0.03);
  border-bottom: 1px solid rgba(255, 255, 255, 0.04);
}

.terminal-dot {
  width: 10px;
  height: 10px;
  border-radius: 50%;
}

.terminal-title {
  margin-left: 8px;
  font-size: 11px;
  color: var(--muted);
  font-family: "SF Mono", "Fira Code", "Cascadia Code", monospace;
}

.code-content {
  padding: 14px 16px;
  font-family: "SF Mono", "Fira Code", "Cascadia Code", "JetBrains Mono", monospace;
  font-size: 14px;
  color: var(--code-text);
  word-break: break-all;
  line-height: 1.6;
  min-height: 48px;
  user-select: all;
}

.input-box {
  width: 100%;
  padding: 12px 14px;
  background: var(--code-bg);
  color: var(--text);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 10px;
  font-size: 14px;
  font-family: "SF Mono", "Fira Code", "Cascadia Code", monospace;
  resize: vertical;
  transition: border-color 0.2s;
  outline: none;
}

.input-box:focus {
  border-color: var(--qq);
}

.input-box::placeholder {
  color: var(--muted);
}

.btn {
  width: 100%;
  padding: 13px;
  border: none;
  border-radius: 10px;
  font-size: 15px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
  font-family: inherit;
  position: relative;
  overflow: hidden;
  background: linear-gradient(135deg, #00b989, #009a6e);
  color: #fff;
  box-shadow: 0 4px 14px rgba(0, 185, 137, 0.3);
  margin-top: 12px;
}

.btn:hover:not(:disabled) {
  transform: translateY(-1px);
  box-shadow: 0 6px 20px rgba(0, 185, 137, 0.4);
}

.btn:active:not(:disabled) {
  transform: translateY(0);
}

.btn:disabled {
  opacity: 0.4;
  cursor: not-allowed;
  box-shadow: none;
}

.btn.copied {
  background: linear-gradient(135deg, #00c853, #00a844);
  box-shadow: 0 4px 14px rgba(0, 200, 83, 0.3);
}

.btn .btn-text { transition: opacity 0.15s; }
.btn .btn-done {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  opacity: 0;
  transition: opacity 0.15s;
}

.btn.copied .btn-text { opacity: 0; }
.btn.copied .btn-done { opacity: 1; }

.footer-tip {
  text-align: center;
  font-size: 12px;
  color: var(--muted);
  margin-top: 24px;
}

@media (max-width: 440px) {
  .card { padding: 24px 18px 22px; }
  .header h1 { font-size: 18px; }
  .tabs button { padding: 8px 16px; font-size: 13px; }
}
</style>