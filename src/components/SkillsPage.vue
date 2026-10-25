<script setup lang="ts">
import { computed, onMounted, onUnmounted, reactive, ref } from 'vue'

type Category = { label: string; skills: string[]; radius: number; x: number; y: number; vx: number; vy: number }
type Sample = { x: number; y: number; time: number }

const arena = ref<HTMLElement | null>(null)
const prefersReducedMotion = ref(false)
const desktopEnabled = ref(false)
const freezePhysics = ref(false)
const selectedCategory = ref<Category | null>(null)
const draggingLabel = ref<string | null>(null)
const categories = reactive<Category[]>([
  { label: 'Frontend & Frameworks', skills: ['AngularJS', 'Angular', 'ReactJS', 'Vue.js', 'Next.js', 'Vite'], radius: 64, x: 0.16, y: 0.22, vx: 0.08, vy: 0.06 },
  { label: 'UI & Interaction', skills: ['Angular Material', 'Materialize UI', 'Bootstrap', 'SCSS', 'HTML', 'CSS', 'Tailwind CSS', 'Shadcn UI', 'Ant Design', 'Framer Motion', 'TanStack Table', 'Ag-Grid', 'dnd-kit', 'React Hook Form', 'Zod', 'Slate.js', 'TipTap'], radius: 58, x: 0.62, y: 0.18, vx: -0.07, vy: 0.08 },
  { label: 'State & Data', skills: ['RxJS', 'BehaviorSubject', 'Zustand', 'Redux', 'SWR', 'TanStack Query'], radius: 52, x: 0.8, y: 0.48, vx: -0.08, vy: -0.05 },
  { label: 'API & Realtime', skills: ['REST API', 'GraphQL Client', 'WebSocket', 'SSE', 'Postman', 'Swagger'], radius: 58, x: 0.3, y: 0.62, vx: 0.09, vy: -0.06 },
  { label: 'Testing & Quality', skills: ['Cypress', 'Playwright', 'Jest'], radius: 48, x: 0.54, y: 0.5, vx: 0.06, vy: 0.09 },
  { label: 'Backend & Cloud', skills: ['Node.js', 'Express', 'MongoDB', 'Mongoose', 'Supabase', 'Cloudflare Pages', 'Cloudflare Workers', 'Cloudflare R2', 'Cloudinary', 'Docker', 'Docker Compose', 'Git', 'GitHub', 'GitLab', 'Jira', 'Trello', 'YouTrack', 'Google Apps Script'], radius: 64, x: 0.72, y: 0.8, vx: -0.07, vy: -0.07 },
])

const MAX_THROW_SPEED = 1.2
let frame = 0
let lastTime = 0
let pointerId = -1
let pointerStart: Sample | null = null
let previousSample: Sample | null = null
let latestSample: Sample | null = null
let dragTarget: HTMLElement | null = null
let observer: ResizeObserver | undefined
let motionQuery: MediaQueryList | undefined
let desktopQuery: MediaQueryList | undefined
const canAnimate = computed(() => desktopEnabled.value && !prefersReducedMotion.value && !document.hidden && !freezePhysics.value)

function dimensions() {
  const rect = arena.value?.getBoundingClientRect()
  return rect ? { width: rect.width, height: rect.height, left: rect.left, top: rect.top } : null
}

function clamp(category: Category, size: { width: number; height: number }) {
  const rx = category.radius / size.width
  const ry = category.radius / size.height
  category.x = Math.min(Math.max(category.x, rx), 1 - rx)
  category.y = Math.min(Math.max(category.y, ry), 1 - ry)
}

function tick(time: number) {
  const size = dimensions()
  if (!size || !canAnimate.value) return
  const dt = Math.min((time - lastTime) / 1000, 0.04)
  lastTime = time
  for (const item of categories) {
    if (item.label === draggingLabel.value) continue
    item.x += item.vx * dt
    item.y += item.vy * dt
    const rx = item.radius / size.width
    const ry = item.radius / size.height
    if (item.x < rx || item.x > 1 - rx) { item.vx *= -1; item.x = Math.min(Math.max(item.x, rx), 1 - rx) }
    if (item.y < ry || item.y > 1 - ry) { item.vy *= -1; item.y = Math.min(Math.max(item.y, ry), 1 - ry) }
  }
  for (let i = 0; i < categories.length; i++) {
    for (let j = i + 1; j < categories.length; j++) {
      const a = categories[i], b = categories[j]
      if (a.label === draggingLabel.value || b.label === draggingLabel.value) continue
      const dx = (b.x - a.x) * size.width, dy = (b.y - a.y) * size.height
      const distance = Math.hypot(dx, dy) || 1
      const minimum = a.radius + b.radius
      if (distance >= minimum) continue
      const nx = dx / distance, ny = dy / distance, push = (minimum - distance) / 2
      a.x -= (nx * push) / size.width; a.y -= (ny * push) / size.height
      b.x += (nx * push) / size.width; b.y += (ny * push) / size.height
      const relative = (a.vx - b.vx) * nx + (a.vy - b.vy) * ny
      if (relative > 0) { a.vx -= relative * nx; a.vy -= relative * ny; b.vx += relative * nx; b.vy += relative * ny }
    }
  }
  categories.forEach((item) => clamp(item, size))
  frame = requestAnimationFrame(tick)
}

function start() {
  cancelAnimationFrame(frame)
  if (!canAnimate.value) return
  lastTime = performance.now()
  frame = requestAnimationFrame(tick)
}

function sample(event: PointerEvent): Sample | null {
  const size = dimensions()
  return size ? { x: event.clientX - size.left, y: event.clientY - size.top, time: event.timeStamp } : null
}

function openCategory(category: Category) {
  selectedCategory.value = category
  freezePhysics.value = true
}

function startDrag(category: Category, event: PointerEvent) {
  if (prefersReducedMotion.value) return
  const point = sample(event)
  if (!point) return
  draggingLabel.value = category.label; pointerId = event.pointerId; pointerStart = point; previousSample = point; latestSample = point
  dragTarget = event.currentTarget as HTMLElement
  dragTarget.setPointerCapture(event.pointerId)
  category.vx = 0; category.vy = 0
}

function moveDrag(event: PointerEvent) {
  if (event.pointerId !== pointerId || !draggingLabel.value) return
  const category = categories.find((item) => item.label === draggingLabel.value)
  const point = sample(event), size = dimensions()
  if (!category || !point || !size) return
  previousSample = latestSample; latestSample = point
  category.x = point.x / size.width; category.y = point.y / size.height
  clamp(category, size)
}

function endDrag(event: PointerEvent) {
  if (event.pointerId !== pointerId || !draggingLabel.value) return
  const category = categories.find((item) => item.label === draggingLabel.value)
  const size = dimensions()
  const moved = pointerStart && latestSample ? Math.hypot(latestSample.x - pointerStart.x, latestSample.y - pointerStart.y) : 0
  if (category && moved < 8) openCategory(category)
  if (category && size && previousSample && latestSample && !prefersReducedMotion.value && moved >= 8) {
    const dt = Math.max((latestSample.time - previousSample.time) / 1000, 1 / 240)
    let vx = ((latestSample.x - previousSample.x) / size.width) / dt
    let vy = ((latestSample.y - previousSample.y) / size.height) / dt
    const speed = Math.hypot(vx, vy)
    if (speed > MAX_THROW_SPEED) { vx = (vx / speed) * MAX_THROW_SPEED; vy = (vy / speed) * MAX_THROW_SPEED }
    category.vx = vx; category.vy = vy
  }
  if (dragTarget?.hasPointerCapture(event.pointerId)) dragTarget.releasePointerCapture(event.pointerId)
  draggingLabel.value = null; pointerId = -1; pointerStart = null; previousSample = null; latestSample = null; dragTarget = null
  start()
}

function closeModal() { selectedCategory.value = null }
function onModalAfterLeave() { freezePhysics.value = false; start() }
function updateMedia() { prefersReducedMotion.value = motionQuery?.matches ?? false; desktopEnabled.value = desktopQuery?.matches ?? false; start() }

onMounted(() => {
  motionQuery = window.matchMedia('(prefers-reduced-motion: reduce)')
  desktopQuery = window.matchMedia('(min-width: 1024px)')
  observer = new ResizeObserver(() => { const size = dimensions(); if (size) categories.forEach((item) => clamp(item, size)) })
  if (arena.value) observer.observe(arena.value)
  motionQuery.addEventListener('change', updateMedia)
  desktopQuery.addEventListener('change', updateMedia)
  document.addEventListener('visibilitychange', start)
  updateMedia()
})

onUnmounted(() => {
  cancelAnimationFrame(frame); observer?.disconnect()
  motionQuery?.removeEventListener('change', updateMedia)
  desktopQuery?.removeEventListener('change', updateMedia)
  document.removeEventListener('visibilitychange', start)
})
</script>

<template>
  <section tabindex="-1" aria-label="Skills" class="relative z-10 flex min-h-0 flex-1 flex-col overflow-visible px-6 py-10 sm:px-12 lg:overflow-hidden lg:px-16 lg:pt-20 lg:pb-0">
    <div class="flex min-h-0 w-full max-w-5xl flex-1 flex-col">
      <p class="text-sm font-semibold tracking-[0.14em] text-violet-300 uppercase">Skills</p>
      <p class="mt-2 text-xs text-slate-500 lg:hidden">Tap an axis to open details</p>
      <div class="relative mt-8 flex min-h-0 flex-1 items-center self-center lg:hidden">
        <svg viewBox="0 0 280 280" class="mx-auto block w-full max-w-[320px] overflow-visible">
          <g fill="none" stroke="rgb(255 255 255 / 0.12)">
            <polygon points="140,72 199,106 199,174 140,208 81,174 81,106" />
            <polygon points="140,96 178,118 178,162 140,184 102,162 102,118" />
            <polygon points="140,118 159,129 159,151 140,162 121,151 121,129" />
            <line x1="140" y1="140" x2="140" y2="56" /><line x1="140" y1="140" x2="213" y2="98" /><line x1="140" y1="140" x2="213" y2="182" />
            <line x1="140" y1="140" x2="140" y2="224" /><line x1="140" y1="140" x2="67" y2="182" /><line x1="140" y1="140" x2="67" y2="98" />
          </g>
          <polygon points="140,78 192,110 177,161 140,187 104,160 92,112" fill="rgb(124 58 237 / 0.22)" stroke="#c4b5fd" stroke-width="2" />
          <g fill="#c4b5fd"><circle cx="140" cy="78" r="3" /><circle cx="192" cy="110" r="3" /><circle cx="177" cy="161" r="3" /><circle cx="140" cy="187" r="3" /><circle cx="104" cy="160" r="3" /><circle cx="92" cy="112" r="3" /></g>
          <g v-for="(category, index) in categories" :key="category.label" class="radar-label" tabindex="0" role="button" :aria-label="`Open ${category.label} skills`" @click="openCategory(category)" @keydown.enter="openCategory(category)" @keydown.space.prevent="openCategory(category)">
            <rect v-if="index === 0" x="105" y="26" width="70" height="26" fill="transparent" />
            <rect v-else-if="index === 1" x="195" y="72" width="85" height="26" fill="transparent" />
            <rect v-else-if="index === 2" x="195" y="174" width="80" height="26" fill="transparent" />
            <rect v-else-if="index === 3" x="95" y="226" width="92" height="26" fill="transparent" />
            <rect v-else-if="index === 4" x="10" y="174" width="70" height="26" fill="transparent" />
            <rect v-else x="0" y="72" width="95" height="26" fill="transparent" />
            <text v-if="index === 0" x="140" y="44" text-anchor="middle">Frontend</text>
            <text v-else-if="index === 1" x="202" y="92" text-anchor="start">UI &amp; Interaction</text>
            <text v-else-if="index === 2" x="202" y="194" text-anchor="start">State &amp; Data</text>
            <text v-else-if="index === 3" x="140" y="244" text-anchor="middle">API &amp; Realtime</text>
            <text v-else-if="index === 4" x="78" y="194" text-anchor="end">Testing</text>
            <text v-else x="78" y="92" text-anchor="end">Backend &amp; Cloud</text>
          </g>
        </svg>
      </div>
      <div ref="arena" class="relative mt-6 hidden min-h-0 flex-1 overflow-hidden lg:block" @pointermove="moveDrag" @pointerup="endDrag" @pointercancel="endDrag">
        <button v-for="category in categories" :key="category.label" type="button" class="skill-bubble absolute flex touch-none select-none items-center justify-center rounded-full border border-white/15 bg-violet-400/10 px-3 text-center text-xs font-semibold text-white shadow-[inset_0_1px_0_rgb(255,255,255,0.16),0_12px_24px_rgb(0,0,0,0.18)] cursor-grab active:cursor-grabbing focus-visible:outline-2 focus-visible:outline-violet-300" :class="{ 'z-10': draggingLabel === category.label }" :style="{ width: `${category.radius * 2}px`, height: `${category.radius * 2}px`, left: `${category.x * 100}%`, top: `${category.y * 100}%`, transform: 'translate(-50%, -50%)' }" @pointerdown="startDrag(category, $event)">{{ category.label }}</button>
      </div>
    </div>

    <Transition name="skills-modal" @after-leave="onModalAfterLeave">
      <div v-if="selectedCategory" class="absolute inset-0 z-50 flex items-center justify-center bg-[#17191e]/80 p-6 backdrop-blur-sm" @click.self="closeModal">
        <section role="dialog" aria-modal="true" :aria-label="selectedCategory.label" class="skills-modal-panel flex max-h-full w-full max-w-xl flex-col rounded-2xl border border-white/10 bg-[#1c1c20] p-6 shadow-2xl shadow-black/40">
          <div class="flex items-start justify-between gap-4">
            <div><p class="text-sm font-semibold tracking-[0.14em] text-violet-300 uppercase">Skills</p><h1 class="mt-2 text-2xl font-semibold text-white">{{ selectedCategory.label }}</h1></div>
            <button type="button" class="rounded-full p-2 text-slate-400 transition hover:bg-white/8 hover:text-white focus-visible:outline-2 focus-visible:outline-violet-300" aria-label="Close skills details" @click="closeModal">×</button>
          </div>
          <ul class="mt-6 min-h-0 overflow-y-auto text-sm leading-7 text-slate-300 sm:grid sm:grid-cols-2 sm:gap-x-8"><li v-for="skill in selectedCategory.skills" :key="skill">{{ skill }}</li></ul>
        </section>
      </div>
    </Transition>
  </section>
</template>

<style scoped>
.radar-label { cursor: pointer; fill: #e2e8f0; font-size: 11px; font-weight: 600; }
.radar-label:focus { outline: none; }
.radar-label:focus text, .radar-label:hover text { fill: #c4b5fd; }
.skill-bubble { will-change: transform; }
.skills-modal-enter-active, .skills-modal-leave-active { transition: opacity 180ms ease; }
.skills-modal-enter-from, .skills-modal-leave-to { opacity: 0; }
.skills-modal-enter-active .skills-modal-panel, .skills-modal-leave-active .skills-modal-panel { transition: transform 180ms ease; }
.skills-modal-enter-from .skills-modal-panel, .skills-modal-leave-to .skills-modal-panel { transform: scale(0.88); }
@media (prefers-reduced-motion: reduce) {
  .skills-modal-enter-active, .skills-modal-leave-active, .skills-modal-enter-active .skills-modal-panel, .skills-modal-leave-active .skills-modal-panel { transition: none; }
}
</style>
