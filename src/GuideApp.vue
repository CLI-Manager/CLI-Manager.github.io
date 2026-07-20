<script setup>
import { computed, nextTick, onBeforeUnmount, onMounted, ref } from 'vue'
import { guideChapters, guideMeta } from './data/featureGuide'

const githubUrl = 'https://github.com/dark-hxx/CLI-Manager'

const query = ref('')
const searchInput = ref(null)
const tocOpen = ref(false)
const navOpen = ref(false)
const currentChapterId = ref(guideChapters[0].id)
const activeSectionId = ref(guideChapters[0].sections[0]?.id ?? '')
const showBackTop = ref(false)
const copiedId = ref('')
const copyFeedback = ref(null)

const sectionCount = guideChapters.reduce((total, chapter) => total + chapter.sections.length, 0)
const currentChapterIndex = computed(() => {
  const index = guideChapters.findIndex((chapter) => chapter.id === currentChapterId.value)
  return index >= 0 ? index : 0
})
const currentChapter = computed(() => guideChapters[currentChapterIndex.value])
const previousChapter = computed(() => guideChapters[currentChapterIndex.value - 1] ?? null)
const nextChapter = computed(() => guideChapters[currentChapterIndex.value + 1] ?? null)

function searchableText(section) {
  return [
    section.title,
    section.purpose,
    section.entry,
    ...(section.before ?? []),
    ...section.steps,
    section.result,
    ...section.notes,
  ]
    .join(' ')
    .toLocaleLowerCase('zh-CN')
}

const searchGroups = computed(() => {
  const keyword = query.value.trim().toLocaleLowerCase('zh-CN')
  if (!keyword) return []

  return guideChapters.flatMap((chapter) => {
    const chapterMatched = [chapter.number, chapter.title, chapter.summary]
      .join(' ')
      .toLocaleLowerCase('zh-CN')
      .includes(keyword)
    const sections = chapterMatched
      ? chapter.sections
      : chapter.sections.filter((section) => searchableText(section).includes(keyword))

    return sections.length ? [{ chapter, sections }] : []
  })
})

const matchedSectionCount = computed(() =>
  searchGroups.value.reduce((total, group) => total + group.sections.length, 0),
)

const iconPaths = {
  compass: '<circle cx="12" cy="12" r="9"/><path d="m16 8-2.5 5.5L8 16l2.5-5.5L16 8Z"/>',
  'folder-tree': '<path d="M3 5h6l2 2h10v12H3Z"/><path d="M7 11h4v4H7zM15 11h2M15 15h2"/>',
  server: '<rect x="3" y="4" width="18" height="6" rx="2"/><rect x="3" y="14" width="18" height="6" rx="2"/><path d="M7 7h.01M7 17h.01M11 7h7M11 17h7"/>',
  terminal: '<rect x="3" y="4" width="18" height="16" rx="2"/><path d="m7 9 3 3-3 3M13 15h4"/>',
  panels: '<rect x="3" y="3" width="18" height="18" rx="2"/><path d="M12 3v18M3 10h9"/>',
  files: '<path d="M6 2h9l4 4v16H6Z"/><path d="M14 2v5h5M3 6v14h3"/>',
  'git-branch': '<circle cx="6" cy="5" r="2"/><circle cx="18" cy="6" r="2"/><circle cx="6" cy="19" r="2"/><path d="M6 7v10M8 7c5 0 4-1 8-1M16 8c0 5-2 7-8 7"/>',
  webhook: '<path d="M12 4v4M4.5 17.5l3-3M19.5 17.5l-3-3"/><circle cx="12" cy="11" r="3"/><path d="M9.5 13.5 7 18h10l-2.5-4.5"/>',
  'bell-ring': '<path d="M18 8a6 6 0 0 0-12 0c0 7-3 7-3 9h18c0-2-3-2-3-9M10 21h4M4 4 2.5 2.5M20 4l1.5-1.5"/>',
  command: '<path d="M18 9a3 3 0 1 0-3-3v12a3 3 0 1 0 3-3H6a3 3 0 1 0 3 3V6a3 3 0 1 0-3 3Z"/>',
  history: '<path d="M3 12a9 9 0 1 0 3-6.7L3 8"/><path d="M3 3v5h5M12 7v5l3 2"/>',
  'git-compare': '<circle cx="7" cy="5" r="2"/><circle cx="17" cy="19" r="2"/><path d="M7 7v12M17 17V5M4 16l3 3 3-3M14 8l3-3 3 3"/>',
  chart: '<path d="M3 3v18h18"/><path d="m7 16 4-5 4 3 5-7"/>',
  sliders: '<path d="M4 7h10M18 7h2M4 17h2M10 17h10"/><circle cx="16" cy="7" r="2"/><circle cx="8" cy="17" r="2"/>',
  'database-backup': '<ellipse cx="12" cy="5" rx="8" ry="3"/><path d="M4 5v6c0 1.7 3.6 3 8 3 1.1 0 2.1-.1 3-.2M4 11v6c0 1.7 3.6 3 8 3"/><path d="m17 17 2 2 3-3M19 13v6"/>',
  settings: '<circle cx="12" cy="12" r="3"/><path d="M19.4 15a1.7 1.7 0 0 0 .3 1.9l.1.1-2.8 2.8-.1-.1a1.7 1.7 0 0 0-1.9-.3 1.7 1.7 0 0 0-1 1.6v.2h-4V21a1.7 1.7 0 0 0-1-1.6 1.7 1.7 0 0 0-1.9.3l-.1.1L4.2 17l.1-.1a1.7 1.7 0 0 0 .3-1.9A1.7 1.7 0 0 0 3 14H2.8v-4H3a1.7 1.7 0 0 0 1.6-1 1.7 1.7 0 0 0-.3-1.9L4.2 7 7 4.2l.1.1a1.7 1.7 0 0 0 1.9.3A1.7 1.7 0 0 0 10 3V2.8h4V3a1.7 1.7 0 0 0 1 1.6 1.7 1.7 0 0 0 1.9-.3l.1-.1L19.8 7l-.1.1a1.7 1.7 0 0 0-.3 1.9 1.7 1.7 0 0 0 1.6 1h.2v4H21a1.7 1.7 0 0 0-1.6 1Z"/>',
}

function iconPath(name) {
  return iconPaths[name] ?? iconPaths.compass
}

function formatInline(value) {
  return value
    .replaceAll('&', '&amp;')
    .replaceAll('<', '&lt;')
    .replaceAll('>', '&gt;')
    .replaceAll('"', '&quot;')
    .replace(/`([^`]+)`/g, '<code>$1</code>')
}

function resolveLocation() {
  const url = new URL(window.location.href)
  const requestedChapter = url.searchParams.get('chapter')
  const hashId = url.hash.slice(1)
  const hashChapter = hashId
    ? guideChapters.find(
        (chapter) => chapter.id === hashId || chapter.sections.some((section) => section.id === hashId),
      )
    : null

  if (hashChapter) {
    return {
      chapterId: hashChapter.id,
      sectionId: hashChapter.id === hashId ? '' : hashId,
    }
  }

  const chapter = guideChapters.find((item) => item.id === requestedChapter) ?? guideChapters[0]
  return { chapterId: chapter.id, sectionId: '' }
}

function updateLocation(chapterId, sectionId = '', mode = 'push') {
  const url = new URL(window.location.href)
  url.searchParams.set('chapter', chapterId)
  url.hash = sectionId
  const nextUrl = `${url.pathname}${url.search}${url.hash}`
  window.history[mode === 'replace' ? 'replaceState' : 'pushState']({}, '', nextUrl)
}

function scrollToReader(sectionId = '', behavior = 'smooth') {
  nextTick(() => {
    const target = sectionId
      ? document.getElementById(sectionId)
      : document.getElementById(currentChapter.value.id) ?? document.getElementById('guide-reader')
    if (!target) return

    const headerHeight =
      Number.parseFloat(
        getComputedStyle(document.documentElement).getPropertyValue('--guide-header-height'),
      ) || 72
    const contentGap = sectionId ? 24 : 16
    const top = window.scrollY + target.getBoundingClientRect().top - headerHeight - contentGap

    window.scrollTo({ top: Math.max(0, top), behavior })
  })
}

function openChapter(chapterId, sectionId = '', options = {}) {
  const chapter = guideChapters.find((item) => item.id === chapterId)
  if (!chapter) return

  currentChapterId.value = chapter.id
  activeSectionId.value = sectionId || chapter.sections[0]?.id || ''
  query.value = ''
  tocOpen.value = false
  navOpen.value = false
  updateLocation(chapter.id, sectionId, options.replace ? 'replace' : 'push')

  if (options.scroll !== false) {
    scrollToReader(sectionId, options.instant ? 'auto' : 'smooth')
  }
}

function openSection(sectionId) {
  if (!currentChapter.value.sections.some((section) => section.id === sectionId)) return

  activeSectionId.value = sectionId
  updateLocation(currentChapter.value.id, sectionId)
  scrollToReader(sectionId)
}

function syncFromLocation({ scroll = false, replace = false } = {}) {
  const location = resolveLocation()
  currentChapterId.value = location.chapterId
  const chapter = guideChapters.find((item) => item.id === location.chapterId)
  activeSectionId.value = location.sectionId || chapter?.sections[0]?.id || ''
  query.value = ''
  tocOpen.value = false
  navOpen.value = false

  const url = new URL(window.location.href)
  if (replace || url.searchParams.get('chapter') !== location.chapterId) {
    updateLocation(location.chapterId, location.sectionId, 'replace')
  }

  if (location.sectionId || scroll) {
    scrollToReader(location.sectionId, 'auto')
  }
}

function clearSearch() {
  query.value = ''
  searchInput.value?.focus()
}

function updateActiveSection() {
  if (query.value) return

  const marker = 150
  let activeId = currentChapter.value.sections[0]?.id ?? ''

  for (const section of currentChapter.value.sections) {
    const element = document.getElementById(section.id)
    if (!element) continue
    if (element.getBoundingClientRect().top <= marker) activeId = section.id
    else break
  }

  activeSectionId.value = activeId
}

function handleScroll() {
  showBackTop.value = window.scrollY > 760
  updateActiveSection()
}

function scrollTop() {
  window.scrollTo({ top: 0, behavior: 'smooth' })
}

let copyFeedbackTimer

async function copySectionLink(id) {
  const url = new URL(window.location.href)
  url.searchParams.set('chapter', currentChapter.value.id)
  url.hash = id

  window.clearTimeout(copyFeedbackTimer)

  try {
    if (!navigator.clipboard?.writeText) throw new Error('Clipboard API unavailable')
    await navigator.clipboard.writeText(url.toString())
    copiedId.value = id
    copyFeedback.value = { type: 'success', message: '章节链接已复制' }
  } catch {
    copiedId.value = ''
    copyFeedback.value = { type: 'error', message: '复制失败，请重试' }
  }

  copyFeedbackTimer = window.setTimeout(() => {
    copiedId.value = ''
    copyFeedback.value = null
  }, 1600)
}

function handleGlobalKeydown(event) {
  const target = event.target
  const isTyping = target instanceof HTMLInputElement || target instanceof HTMLTextAreaElement

  if (event.key === '/' && !isTyping) {
    event.preventDefault()
    searchInput.value?.focus()
  }

  if (event.key === 'Escape') {
    if (navOpen.value) navOpen.value = false
    else if (tocOpen.value) tocOpen.value = false
    else if (query.value) clearSearch()
  }
}

function handlePopState() {
  syncFromLocation({ scroll: true })
}

onMounted(() => {
  window.addEventListener('scroll', handleScroll, { passive: true })
  window.addEventListener('keydown', handleGlobalKeydown)
  window.addEventListener('popstate', handlePopState)
  handleScroll()
  syncFromLocation({ replace: true })
  nextTick(updateActiveSection)
})

onBeforeUnmount(() => {
  window.removeEventListener('scroll', handleScroll)
  window.removeEventListener('keydown', handleGlobalKeydown)
  window.removeEventListener('popstate', handlePopState)
  window.clearTimeout(copyFeedbackTimer)
})
</script>

<template>
  <div class="guide-shell">
    <header class="guide-topbar">
      <div class="guide-topbar-inner">
        <a class="brand" href="./" aria-label="返回 CLI-Manager 首页">
          <img class="brand-mark" src="/assets/brand/cli-manager-logo.png" alt="" aria-hidden="true" />
          <span class="brand-name">CLI<span>Manager</span></span>
        </a>

        <nav class="guide-desktop-nav" aria-label="主导航">
          <a href="./#features">功能</a>
          <a href="./#showcase">产品</a>
          <a href="./#workflow">工作流</a>
          <a href="./#download">下载</a>
          <a class="active" href="./guide.html">手册</a>
        </nav>

        <div class="guide-top-actions">
          <a class="guide-icon-button guide-github-button" :href="githubUrl" target="_blank" rel="noreferrer" title="打开 GitHub" aria-label="打开 GitHub">
            <svg viewBox="0 0 24 24" aria-hidden="true"><path d="M12 2a10 10 0 0 0-3.16 19.49c.5.09.68-.22.68-.48v-1.87c-2.78.6-3.37-1.18-3.37-1.18-.45-1.16-1.11-1.47-1.11-1.47-.91-.62.07-.61.07-.61 1 .07 1.53 1.03 1.53 1.03.9 1.53 2.35 1.09 2.92.83.09-.65.35-1.09.64-1.34-2.22-.25-4.56-1.11-4.56-4.94 0-1.09.39-1.98 1.03-2.68-.1-.25-.45-1.27.1-2.64 0 0 .84-.27 2.75 1.02A9.6 9.6 0 0 1 12 6.82a9.6 9.6 0 0 1 2.5.34c1.91-1.29 2.75-1.02 2.75-1.02.55 1.37.2 2.39.1 2.64.64.7 1.03 1.59 1.03 2.68 0 3.84-2.34 4.68-4.57 4.93.36.31.68.92.68 1.86V21c0 .27.18.58.69.48A10 10 0 0 0 12 2Z"/></svg>
          </a>
          <button class="guide-icon-button guide-toc-toggle" type="button" :aria-expanded="tocOpen" title="打开章节目录" aria-label="打开章节目录" @click="tocOpen = !tocOpen; navOpen = false">
            <svg viewBox="0 0 24 24" aria-hidden="true"><path d="M4 5h6v14H4zM14 5h6v14h-6zM7 8h.01M17 8h.01M7 12h.01M17 12h.01"/></svg>
          </button>
          <button class="guide-icon-button guide-nav-toggle" type="button" :aria-expanded="navOpen" title="打开主导航" aria-label="打开主导航" @click="navOpen = !navOpen; tocOpen = false">
            <svg viewBox="0 0 24 24" aria-hidden="true"><path d="M4 6h16M4 12h16M4 18h16"/></svg>
          </button>
        </div>
      </div>

      <nav class="guide-mobile-nav" :class="{ open: navOpen }" aria-label="移动端主导航">
        <a href="./#features" @click="navOpen = false">功能</a>
        <a href="./#showcase" @click="navOpen = false">产品</a>
        <a href="./#workflow" @click="navOpen = false">工作流</a>
        <a href="./#download" @click="navOpen = false">下载</a>
        <a class="active" href="./guide.html?chapter=getting-started" @click="navOpen = false">手册</a>
      </nav>
    </header>

    <main>
      <section class="guide-hero">
        <div class="guide-wide-container">
          <span class="guide-eyebrow">Product manual · {{ guideMeta.version }}</span>
          <h1>{{ guideMeta.title }}</h1>
          <p>{{ guideMeta.description }}</p>

          <div class="guide-stats" aria-label="手册规模">
            <span><b>{{ guideChapters.length }}</b> 个主章节</span>
            <span><b>{{ sectionCount }}</b> 个操作主题</span>
            <span><b>{{ guideMeta.updatedAt }}</b> 最近更新</span>
          </div>

          <label class="guide-search">
            <svg viewBox="0 0 24 24" aria-hidden="true"><circle cx="11" cy="11" r="7"/><path d="m20 20-4-4"/></svg>
            <input ref="searchInput" v-model="query" type="search" placeholder="搜索功能、入口、操作步骤或故障关键词" aria-label="搜索功能手册" />
            <button v-if="query" type="button" title="清空搜索" aria-label="清空搜索" @click="clearSearch">
              <svg viewBox="0 0 24 24" aria-hidden="true"><path d="m6 6 12 12M18 6 6 18"/></svg>
            </button>
          </label>
        </div>
      </section>

      <div id="guide-reader" class="guide-wide-container guide-layout">
        <div class="guide-toc-backdrop" :class="{ visible: tocOpen }" @click="tocOpen = false"></div>
        <aside class="guide-sidebar" :class="{ open: tocOpen }">
          <div class="guide-sidebar-head">
            <div>
              <span>Manual index</span>
              <b>章节目录</b>
            </div>
            <button type="button" title="关闭章节目录" aria-label="关闭章节目录" @click="tocOpen = false">
              <svg viewBox="0 0 24 24" aria-hidden="true"><path d="m6 6 12 12M18 6 6 18"/></svg>
            </button>
          </div>

          <nav aria-label="手册章节">
            <button
              v-for="chapter in guideChapters"
              :key="chapter.id"
              type="button"
              :class="{ active: currentChapter.id === chapter.id }"
              @click="openChapter(chapter.id)"
            >
              <span>{{ chapter.number }}</span>
              <b>{{ chapter.title }}</b>
            </button>
          </nav>
        </aside>

        <div class="guide-content">
          <template v-if="query">
            <div class="guide-search-summary" role="status">
              <span>搜索“{{ query }}”</span>
              <b>找到 {{ matchedSectionCount }} 个操作主题</b>
            </div>

            <div v-if="searchGroups.length" class="guide-search-results">
              <section v-for="group in searchGroups" :key="group.chapter.id" class="guide-search-group">
                <button class="guide-search-chapter" type="button" @click="openChapter(group.chapter.id)">
                  <span>{{ group.chapter.number }}</span>
                  <div><b>{{ group.chapter.title }}</b><small>{{ group.sections.length }} 个匹配主题</small></div>
                  <svg viewBox="0 0 24 24" aria-hidden="true"><path d="m9 18 6-6-6-6"/></svg>
                </button>
                <div class="guide-search-items">
                  <button v-for="section in group.sections" :key="section.id" type="button" @click="openChapter(group.chapter.id, section.id)">
                    <div><h3>{{ section.title }}</h3><p>{{ section.purpose }}</p></div>
                    <span>{{ section.entry }}</span>
                  </button>
                </div>
              </section>
            </div>

            <div v-else class="guide-empty">
              <svg viewBox="0 0 24 24" aria-hidden="true"><circle cx="11" cy="11" r="7"/><path d="m20 20-4-4M8 11h6"/></svg>
              <h2>没有找到相关内容</h2>
              <p>尝试搜索“Hook”“Worktree”“历史会话”“备份”或具体设置名称。</p>
              <button type="button" @click="clearSearch">清空搜索</button>
            </div>
          </template>

          <template v-else>
            <article :id="currentChapter.id" :key="currentChapter.id" class="guide-chapter">
              <header class="guide-chapter-head">
                <div class="guide-chapter-mark">
                  <svg viewBox="0 0 24 24" aria-hidden="true" v-html="iconPath(currentChapter.icon)"></svg>
                </div>
                <div>
                  <span>Chapter {{ currentChapter.number }} · {{ currentChapterIndex + 1 }} / {{ guideChapters.length }}</span>
                  <h2>{{ currentChapter.title }}</h2>
                  <p>{{ currentChapter.summary }}</p>
                </div>
              </header>

              <figure v-if="currentChapter.image" class="guide-figure">
                <img :src="currentChapter.image.src" :alt="currentChapter.image.alt" />
                <figcaption>{{ currentChapter.image.caption }}</figcaption>
              </figure>

              <div class="guide-procedures">
                <section v-for="section in currentChapter.sections" :id="section.id" :key="section.id" class="guide-procedure">
                  <header>
                    <div>
                      <span>操作说明</span>
                      <h3>{{ section.title }}</h3>
                    </div>
                    <button type="button" :class="{ copied: copiedId === section.id }" title="复制此节链接" aria-label="复制此节链接" @click="copySectionLink(section.id)">
                      <svg v-if="copiedId === section.id" viewBox="0 0 24 24" aria-hidden="true"><path d="m5 12 4 4L19 6"/></svg>
                      <svg v-else viewBox="0 0 24 24" aria-hidden="true"><path d="M10 13a5 5 0 0 0 7.1.1l2-2a5 5 0 0 0-7.1-7.1l-1.1 1.1M14 11a5 5 0 0 0-7.1-.1l-2 2A5 5 0 0 0 12 20l1.1-1.1"/></svg>
                    </button>
                  </header>

                  <p class="guide-purpose" v-html="formatInline(section.purpose)"></p>

                  <dl class="guide-meta">
                    <div>
                      <dt>功能入口</dt>
                      <dd v-html="formatInline(section.entry)"></dd>
                    </div>
                    <div v-if="section.before?.length">
                      <dt>开始之前</dt>
                      <dd><span v-for="item in section.before" :key="item" v-html="formatInline(item)"></span></dd>
                    </div>
                  </dl>

                  <div class="guide-steps">
                    <h4>操作步骤</h4>
                    <ol>
                      <li v-for="(step, index) in section.steps" :key="step">
                        <span>{{ String(index + 1).padStart(2, '0') }}</span>
                        <p v-html="formatInline(step)"></p>
                      </li>
                    </ol>
                  </div>

                  <div class="guide-outcome">
                    <svg viewBox="0 0 24 24" aria-hidden="true"><path d="m5 12 4 4L19 6"/></svg>
                    <div><b>完成结果</b><p v-html="formatInline(section.result)"></p></div>
                  </div>

                  <div class="guide-notes">
                    <b>注意事项</b>
                    <ul><li v-for="note in section.notes" :key="note" v-html="formatInline(note)"></li></ul>
                  </div>
                </section>
              </div>
            </article>

            <nav class="guide-chapter-pagination" aria-label="章节翻页">
              <button type="button" :disabled="!previousChapter" @click="previousChapter && openChapter(previousChapter.id)">
                <svg viewBox="0 0 24 24" aria-hidden="true"><path d="m15 18-6-6 6-6"/></svg>
                <span><small>上一章节</small><b>{{ previousChapter ? `${previousChapter.number} ${previousChapter.title}` : '已是第一章' }}</b></span>
              </button>
              <button class="next" type="button" :disabled="!nextChapter" @click="nextChapter && openChapter(nextChapter.id)">
                <span><small>下一章节</small><b>{{ nextChapter ? `${nextChapter.number} ${nextChapter.title}` : '已是最后一章' }}</b></span>
                <svg viewBox="0 0 24 24" aria-hidden="true"><path d="m9 18 6-6-6-6"/></svg>
              </button>
            </nav>
          </template>
        </div>

        <aside v-if="!query" class="guide-section-outline">
          <div class="guide-section-outline-head">
            <span>Page outline</span>
            <b>本章大纲</b>
            <small>{{ currentChapter.sections.length }} 个操作主题</small>
          </div>
          <nav aria-label="当前章节大纲">
            <button
              v-for="(section, index) in currentChapter.sections"
              :key="section.id"
              type="button"
              :class="{ active: activeSectionId === section.id }"
              @click="openSection(section.id)"
            >
              <span>{{ String(index + 1).padStart(2, '0') }}</span>
              <b>{{ section.title }}</b>
            </button>
          </nav>
        </aside>
      </div>
    </main>

    <footer class="guide-footer">
      <div class="guide-wide-container">
        <div>
          <a class="brand" href="./">
            <img class="brand-mark" src="/assets/brand/cli-manager-logo.png" alt="" aria-hidden="true" />
            <span class="brand-name">CLI<span>Manager</span></span>
          </a>
          <p>让 AI CLI 工作流清晰、从容、高效。</p>
        </div>
        <div class="guide-footer-links">
          <a href="./">返回首页</a>
          <a :href="githubUrl" target="_blank" rel="noreferrer">GitHub</a>
          <a :href="`${githubUrl}/releases`" target="_blank" rel="noreferrer">更新日志</a>
        </div>
      </div>
    </footer>

    <button class="guide-back-top" :class="{ visible: showBackTop }" type="button" title="返回顶部" aria-label="返回顶部" @click="scrollTop">
      <svg viewBox="0 0 24 24" aria-hidden="true"><path d="m6 15 6-6 6 6"/></svg>
    </button>

    <Transition name="guide-toast">
      <div v-if="copyFeedback" class="guide-copy-toast" :class="`is-${copyFeedback.type}`" role="status">
        <svg v-if="copyFeedback.type === 'success'" viewBox="0 0 24 24" aria-hidden="true"><path d="m5 12 4 4L19 6"/></svg>
        <svg v-else viewBox="0 0 24 24" aria-hidden="true"><path d="m7 7 10 10M17 7 7 17"/></svg>
        <span>{{ copyFeedback.message }}</span>
      </div>
    </Transition>
  </div>
</template>
