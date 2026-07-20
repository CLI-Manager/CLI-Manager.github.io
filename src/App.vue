<script setup>
import { computed, onBeforeUnmount, onMounted, ref } from 'vue'

const githubUrl = 'https://github.com/dark-hxx/CLI-Manager'
const releaseUrl = `${githubUrl}/releases/latest`

const menuOpen = ref(false)
const scrolled = ref(false)
const activeShowcase = ref(0)
const showBackTop = ref(false)

const showcases = [
  {
    kicker: 'Terminal Workspace',
    title: '多项目、多终端，一处清晰掌控',
    description: '从项目树到自由分屏，每个 AI CLI 会话都有自己的空间、状态和上下文。',
    image: '/assets/product/terminal-split.png',
    alt: 'CLI-Manager 多终端分屏界面',
    chips: ['自由分屏', 'Tab 跨面板拖拽', '多 Shell 支持'],
  },
  {
    kicker: 'Session Memory',
    title: '每一次对话，都成为可检索的记忆',
    description: '集中浏览 Claude Code 与 Codex 历史会话，搜索、收藏、查看 Diff，并精准跳回触发消息。',
    image: '/assets/product/history.png',
    alt: 'CLI-Manager 会话历史界面',
    chips: ['统一历史', 'Diff 回看', 'Prompt 复用'],
  },
  {
    kicker: 'Usage Intelligence',
    title: '看见 Token 去向，也看见效率提升',
    description: '从实时会话到长期趋势，用清晰的图表理解成本、模型、项目与活跃时间。',
    image: '/assets/product/analytics.png',
    alt: 'CLI-Manager 用量分析看板',
    chips: ['费用估算', '趋势洞察', '项目对比'],
  },
]

const currentShowcase = computed(() => showcases[activeShowcase.value])

const featureCards = [
  {
    number: '01',
    icon: 'bell',
    title: '不再守着终端等审批',
    text: '权限请求、任务完成与失败状态实时提醒，点击通知直接回到对应会话。',
    tone: 'cyan',
  },
  {
    number: '02',
    icon: 'chart',
    title: '用量与费用，实时有数',
    text: '追踪 Input、Output 与缓存 Token，自动估算模型费用，成本变化一目了然。',
    tone: 'blue',
  },
  {
    number: '03',
    icon: 'history',
    title: '会话历史不再散落',
    text: 'Claude Code 与 Codex 会话集中管理，支持搜索、标签、收藏与代码变更回看。',
    tone: 'violet',
  },
  {
    number: '04',
    icon: 'split',
    title: '为并行工作而生',
    text: '自由拆分终端面板，拖拽 Tab 跨面板移动，在多个项目间保持专注。',
    tone: 'green',
  },
  {
    number: '05',
    icon: 'switch',
    title: '项目级供应商切换',
    text: '不同项目使用不同 Claude 后端，无需反复修改环境变量和配置文件。',
    tone: 'orange',
  },
  {
    number: '06',
    icon: 'command',
    title: '高频操作，一键抵达',
    text: '通过命令面板和三级命令模板，快速启动项目、执行命令、复用工作流。',
    tone: 'rose',
  },
]

const platformCards = [
  { name: 'Windows', detail: '完整支持', mark: '⊞', primary: true },
  { name: 'macOS', detail: '实验性支持', mark: '●', primary: false },
  { name: 'Linux', detail: '实验性支持', mark: '⌁', primary: false },
]

function iconPath(name) {
  const icons = {
    bell: '<path d="M18 8a6 6 0 0 0-12 0c0 7-3 7-3 9h18c0-2-3-2-3-9"/><path d="M10 21h4"/>',
    chart: '<path d="M3 3v18h18"/><path d="m7 16 4-5 4 3 5-7"/>',
    history: '<path d="M3 12a9 9 0 1 0 3-6.7L3 8"/><path d="M3 3v5h5"/><path d="M12 7v5l3 2"/>',
    split: '<rect width="18" height="18" x="3" y="3" rx="2"/><path d="M12 3v18M3 9h9"/>',
    switch: '<path d="m17 3 4 4-4 4"/><path d="M3 7h18"/><path d="m7 21-4-4 4-4"/><path d="M21 17H3"/>',
    command: '<path d="M18 9a3 3 0 1 0-3-3v12a3 3 0 1 0 3-3H6a3 3 0 1 0 3 3V6a3 3 0 1 0-3 3Z"/>',
  }
  return icons[name]
}

function closeMenu() {
  menuOpen.value = false
}

function handleScroll() {
  scrolled.value = window.scrollY > 24
  showBackTop.value = window.scrollY > 720
}

function scrollTop() {
  window.scrollTo({ top: 0, behavior: 'smooth' })
}

let revealObserver

onMounted(() => {
  window.addEventListener('scroll', handleScroll, { passive: true })
  handleScroll()

  revealObserver = new IntersectionObserver(
    (entries) => {
      entries.forEach((entry) => {
        if (entry.isIntersecting) entry.target.classList.add('is-visible')
      })
    },
    { threshold: 0.12 },
  )

  document.querySelectorAll('[data-reveal]').forEach((el) => revealObserver.observe(el))
})

onBeforeUnmount(() => {
  window.removeEventListener('scroll', handleScroll)
  revealObserver?.disconnect()
})
</script>

<template>
  <div class="site-shell">
    <header class="topbar" :class="{ 'is-scrolled': scrolled }">
      <div class="container nav-wrap">
        <a class="brand" href="#top" aria-label="CLI-Manager 首页" @click="closeMenu">
          <img class="brand-mark" src="/assets/brand/cli-manager-logo.png" alt="" aria-hidden="true" />
          <span class="brand-name">CLI<span>Manager</span></span>
        </a>

        <nav class="desktop-nav" aria-label="主导航">
          <a href="#features">功能</a>
          <a href="#showcase">产品</a>
          <a href="#workflow">工作流</a>
          <a href="#download">下载</a>
          <a href="./guide.html">手册</a>
        </nav>

        <div class="nav-actions">
          <a class="github-link" :href="githubUrl" target="_blank" rel="noreferrer" aria-label="前往 GitHub">
            <svg viewBox="0 0 24 24" aria-hidden="true"><path d="M12 2a10 10 0 0 0-3.16 19.49c.5.09.68-.22.68-.48v-1.87c-2.78.6-3.37-1.18-3.37-1.18-.45-1.16-1.11-1.47-1.11-1.47-.91-.62.07-.61.07-.61 1 .07 1.53 1.03 1.53 1.03.9 1.53 2.35 1.09 2.92.83.09-.65.35-1.09.64-1.34-2.22-.25-4.56-1.11-4.56-4.94 0-1.09.39-1.98 1.03-2.68-.1-.25-.45-1.27.1-2.64 0 0 .84-.27 2.75 1.02A9.6 9.6 0 0 1 12 6.82a9.6 9.6 0 0 1 2.5.34c1.91-1.29 2.75-1.02 2.75-1.02.55 1.37.2 2.39.1 2.64.64.7 1.03 1.59 1.03 2.68 0 3.84-2.34 4.68-4.57 4.93.36.31.68.92.68 1.86V21c0 .27.18.58.69.48A10 10 0 0 0 12 2Z"/></svg>
            <span>GitHub</span>
          </a>
          <a class="button button-small" :href="releaseUrl" target="_blank" rel="noreferrer">立即下载</a>
        </div>

        <button class="menu-toggle" type="button" :aria-expanded="menuOpen" aria-label="打开导航菜单" @click="menuOpen = !menuOpen">
          <span></span><span></span>
        </button>
      </div>

      <div class="mobile-menu" :class="{ 'is-open': menuOpen }">
        <nav aria-label="移动端导航">
          <a href="#features" @click="closeMenu">功能 <span>01</span></a>
          <a href="#showcase" @click="closeMenu">产品 <span>02</span></a>
          <a href="#workflow" @click="closeMenu">工作流 <span>03</span></a>
          <a href="#download" @click="closeMenu">下载 <span>04</span></a>
          <a href="./guide.html" @click="closeMenu">功能手册 <span>05</span></a>
        </nav>
        <a class="button mobile-download" :href="releaseUrl" target="_blank" rel="noreferrer">下载 CLI-Manager</a>
      </div>
    </header>

    <main id="top">
      <section class="hero">
        <div class="hero-orb hero-orb-one"></div>
        <div class="hero-orb hero-orb-two"></div>
        <div class="hero-grid"></div>
        <div class="container hero-content">
          <div class="hero-copy" data-reveal>
            <a class="eyebrow" :href="releaseUrl" target="_blank" rel="noreferrer">
              <span class="pulse-dot"></span>
              v1.2.9 已发布
              <span class="eyebrow-arrow">→</span>
            </a>
            <h1>让 AI CLI 工作流<br /><span>清晰、从容、<br class="mobile-break" />高效。</span></h1>
            <p class="hero-lead">
              为 Claude Code 与 Codex CLI 深度优化的跨平台工作台。<br class="desktop-only" />
              管理终端、会话、用量与供应商，把复杂留给工具，把专注还给创造。
            </p>
            <div class="hero-actions">
              <a class="button button-primary" :href="releaseUrl" target="_blank" rel="noreferrer">
                <svg viewBox="0 0 24 24" aria-hidden="true"><path d="M12 3v12m0 0 5-5m-5 5-5-5M5 21h14"/></svg>
                下载 Windows 版
              </a>
              <a class="button button-ghost" :href="githubUrl" target="_blank" rel="noreferrer">
                查看开源项目
                <span>↗</span>
              </a>
            </div>
            <div class="hero-meta">
              <span><i></i> 免费开源</span>
              <span><i></i> 数据本地存储</span>
              <span><i></i> Windows / macOS / Linux</span>
            </div>
          </div>

          <div class="hero-visual" data-reveal>
            <div class="visual-halo"></div>
            <div class="app-window hero-window">
              <div class="window-screen hero-screen">
                <img src="/assets/product/hero-app.png" alt="CLI-Manager 产品主界面" />
                <span class="screen-scan" aria-hidden="true"></span>
              </div>
            </div>
          </div>
        </div>

        <div class="container tools-strip" data-reveal>
          <span>专为你的 AI 开发栈而生</span>
          <div class="tool-list">
            <b><i class="tool-symbol claude">✳</i> Claude Code</b>
            <b><i class="tool-symbol codex">◎</i> Codex CLI</b>
            <b><i class="tool-symbol shell">›_</i> PowerShell</b>
            <b><i class="tool-symbol terminal">⌁</i> Bash / Zsh</b>
          </div>
        </div>
      </section>

      <section id="features" class="section features-section">
        <div class="container">
          <div class="section-head split-head" data-reveal>
            <div>
              <span class="section-kicker">Less friction, more flow</span>
              <h2>不只是管理终端。<br />而是管理整个 AI 工作流。</h2>
            </div>
            <p>从任务开始到会话复盘，把原本散落在多个窗口、配置与日志里的信息，收进一个安静而强大的工作空间。</p>
          </div>

          <div class="feature-grid">
            <article v-for="card in featureCards" :key="card.number" class="feature-card" :class="`tone-${card.tone}`" data-reveal>
              <div class="feature-top">
                <span class="feature-icon">
                  <svg viewBox="0 0 24 24" aria-hidden="true" v-html="iconPath(card.icon)"></svg>
                </span>
                <span class="feature-number">{{ card.number }}</span>
              </div>
              <h3>{{ card.title }}</h3>
              <p>{{ card.text }}</p>
              <span class="feature-line"></span>
            </article>
          </div>
        </div>
      </section>

      <section id="showcase" class="section showcase-section">
        <div class="container">
          <div class="section-head center-head" data-reveal>
            <span class="section-kicker">One calm workspace</span>
            <h2>复杂能力，简单呈现</h2>
            <p>真正的效率工具，不该制造新的负担。</p>
          </div>

          <div class="showcase-tabs" role="tablist" aria-label="产品功能预览" data-reveal>
            <button
              v-for="(item, index) in showcases"
              :key="item.kicker"
              type="button"
              role="tab"
              :aria-selected="activeShowcase === index"
              :class="{ active: activeShowcase === index }"
              @click="activeShowcase = index"
            >
              <span>0{{ index + 1 }}</span>{{ item.kicker }}
            </button>
          </div>

          <div class="showcase-stage" data-reveal>
            <div class="showcase-copy">
              <span class="showcase-kicker">{{ currentShowcase.kicker }}</span>
              <h3>{{ currentShowcase.title }}</h3>
              <p>{{ currentShowcase.description }}</p>
              <div class="showcase-chips">
                <span v-for="chip in currentShowcase.chips" :key="chip"><i></i>{{ chip }}</span>
              </div>
              <a href="./guide.html" class="text-link">查看详细操作手册 <span>→</span></a>
            </div>
            <div class="showcase-visual">
              <div class="app-window product-window">
                <div class="window-bar compact">
                  <div class="window-dots"><i></i><i></i><i></i></div>
                  <div class="window-title">CLI-Manager</div>
                </div>
                <div class="window-screen">
                  <Transition name="fade" mode="out-in">
                    <img :key="currentShowcase.image" :src="currentShowcase.image" :alt="currentShowcase.alt" />
                  </Transition>
                </div>
              </div>
            </div>
          </div>
        </div>
      </section>

      <section id="workflow" class="section workflow-section">
        <div class="container workflow-layout">
          <div class="workflow-copy" data-reveal>
            <span class="section-kicker">Stay in the flow</span>
            <h2>从启动项目，<br />到复盘每一次创造。</h2>
            <p>CLI-Manager 沿着你的真实开发节奏工作，不改变习惯，只让每一步更顺畅。</p>
            <a class="button button-dark" href="./guide.html">打开功能操作手册 <span>→</span></a>
          </div>
          <div class="workflow-steps">
            <article data-reveal>
              <span>01</span>
              <div><h3>打开项目</h3><p>统一组织项目与终端，用命令模板快速进入工作状态。</p></div>
              <i>+</i>
            </article>
            <article data-reveal>
              <span>02</span>
              <div><h3>并行执行</h3><p>自由分屏运行 Claude Code、Codex 和本地命令，状态始终可见。</p></div>
              <i>+</i>
            </article>
            <article data-reveal>
              <span>03</span>
              <div><h3>及时响应</h3><p>审批与完成通知主动抵达，不再把注意力浪费在等待上。</p></div>
              <i>+</i>
            </article>
            <article data-reveal>
              <span>04</span>
              <div><h3>沉淀复盘</h3><p>回看历史会话、代码 Diff 与用量趋势，让经验持续复利。</p></div>
              <i>✓</i>
            </article>
          </div>
        </div>
      </section>

      <section class="section local-section">
        <div class="container local-card" data-reveal>
          <div class="local-glow"></div>
          <div class="local-copy">
            <span class="lock-icon">
              <svg viewBox="0 0 24 24"><rect x="4" y="10" width="16" height="11" rx="2"/><path d="M8 10V7a4 4 0 0 1 8 0v3"/></svg>
            </span>
            <div>
              <span class="section-kicker light">Local-first by design</span>
              <h2>你的项目与会话，<br />始终留在你的设备里。</h2>
            </div>
          </div>
          <div class="local-points">
            <span><i>✓</i> 本地 SQLite 存储</span>
            <span><i>✓</i> 无需上传项目代码</span>
            <span><i>✓</i> 开源透明，可自主审查</span>
          </div>
        </div>
      </section>

      <section id="download" class="section download-section">
        <div class="download-orb"></div>
        <div class="container">
          <div class="download-head" data-reveal>
            <span class="section-kicker">Start your focused workflow</span>
            <h2>让下一次 AI 协作，<br />从容一点。</h2>
            <p>免费、开源，几分钟即可开始。</p>
          </div>

          <div class="platform-grid" data-reveal>
            <a
              v-for="platform in platformCards"
              :key="platform.name"
              class="platform-card"
              :class="{ primary: platform.primary }"
              :href="releaseUrl"
              target="_blank"
              rel="noreferrer"
            >
              <span class="platform-mark">{{ platform.mark }}</span>
              <span class="platform-info"><b>{{ platform.name }}</b><small>{{ platform.detail }}</small></span>
              <span class="platform-action">{{ platform.primary ? '下载' : '查看' }} <i>↗</i></span>
            </a>
          </div>

          <p class="download-note" data-reveal>当前 Windows 版本经过完整测试，macOS 与 Linux 处于实验性支持阶段。</p>
        </div>
      </section>
    </main>

    <footer class="footer">
      <div class="container footer-top">
        <div class="footer-brand">
          <a class="brand" href="#top">
            <img class="brand-mark" src="/assets/brand/cli-manager-logo.png" alt="" aria-hidden="true" />
            <span class="brand-name">CLI<span>Manager</span></span>
          </a>
          <p>让 AI CLI 工作流清晰、从容、高效。</p>
        </div>
        <div class="footer-links">
          <div><b>产品</b><a href="#features">核心功能</a><a href="#showcase">产品预览</a><a href="#download">下载</a></div>
          <div><b>资源</b><a :href="githubUrl" target="_blank" rel="noreferrer">GitHub</a><a href="./guide.html">功能手册</a><a :href="`${githubUrl}/releases`" target="_blank" rel="noreferrer">更新日志</a></div>
        </div>
      </div>
      <div class="container footer-bottom">
        <span>© 2026 CLI-Manager · AGPL-3.0-or-later</span>
        <span>Built for focused creators.</span>
      </div>
    </footer>

    <button class="back-top" :class="{ visible: showBackTop }" type="button" aria-label="返回顶部" @click="scrollTop">↑</button>
  </div>
</template>
