<script setup lang="ts">
import { BadgeCheck, Braces, BriefcaseBusiness, Code2, GraduationCap, UserRound } from '@lucide/vue'
import { computed, onMounted, onUnmounted, reactive, ref } from 'vue'
import CertificationPage from './CertificationPage.vue'
import EducationPage from './EducationPage.vue'
import ExperiencePage from './ExperiencePage.vue'
import ProjectsPage from './ProjectsPage.vue'
import SkillsPage from './SkillsPage.vue'

type View = 'home' | 'experience' | 'education' | 'certification' | 'skills' | 'projects'

const trailZone = ref<HTMLElement | null>(null)
const particles = reactive<{ id: number; x: number; y: number }[]>([])
const spotlight = reactive({ x: -9999, y: -9999 })
const glossPosition = reactive({ x: 50, y: 50 })
const glossAngle = ref('125deg')
const canAnimate = typeof matchMedia === 'function' && matchMedia('(pointer: fine)').matches && !matchMedia('(prefers-reduced-motion: reduce)').matches
const activeView = ref<View>('home')
const viewDirection = ref<'forward' | 'back'>('forward')
const viewTransition = computed(() => `view-${viewDirection.value}`)
let lastSpawn = 0
let nextParticleId = 0

function navigateTo(view: View) {
  if (view === activeView.value) return
  viewDirection.value = view === 'home' ? 'back' : 'forward'
  activeView.value = view
}
function hideLeavingCanvas(element: Element) { const canvas = element.querySelector('canvas'); if (canvas) canvas.style.visibility = 'hidden' }
function focusView(element: Element) { ;(element as HTMLElement).focus() }
function addBubble(event: PointerEvent) {
  const now = performance.now()
  if (now - lastSpawn < 25 || !trailZone.value) return
  lastSpawn = now
  const bounds = trailZone.value.getBoundingClientRect()
  spotlight.x = event.clientX - bounds.left; spotlight.y = event.clientY - bounds.top
  glossPosition.x = 100 - (spotlight.x / bounds.width) * 100; glossPosition.y = 100 - (spotlight.y / bounds.height) * 100
  glossAngle.value = `${Math.round(Math.atan2(spotlight.y - bounds.height / 2, spotlight.x - bounds.width / 2) * (180 / Math.PI) + 90)}deg`
  particles.push({ id: nextParticleId++, x: spotlight.x, y: spotlight.y })
  if (particles.length > 40) particles.shift()
}
function removeBubble(id: number) { const index = particles.findIndex((particle) => particle.id === id); if (index !== -1) particles.splice(index, 1) }
onMounted(() => { if (canAnimate) trailZone.value?.addEventListener('pointermove', addBubble, { passive: true }) })
onUnmounted(() => { trailZone.value?.removeEventListener('pointermove', addBubble) })
</script>

<template>
  <div class="min-h-svh bg-[#17191e] lg:bg-[#717b90] lg:p-10">
    <div ref="trailZone" class="relative mx-auto flex h-svh w-full max-w-7xl flex-col overflow-hidden bg-[#17191e] text-white lg:h-auto lg:min-h-[calc(100svh-5rem)] lg:flex-row lg:justify-normal lg:rounded-[3rem] lg:border lg:border-white/10 lg:shadow-2xl lg:shadow-black/35">
      <div v-if="canAnimate" class="surface-gloss pointer-events-none absolute inset-0 z-40" :style="{ '--gloss-x': `${glossPosition.x}%`, '--gloss-y': `${glossPosition.y}%`, '--gloss-angle': glossAngle }" aria-hidden="true"></div>
      <div class="pointer-events-none absolute inset-0 z-50" aria-hidden="true"><span v-for="particle in particles" :key="particle.id" class="trail-bubble absolute" :style="{ left: `${particle.x}px`, top: `${particle.y}px` }" @animationend="removeBubble(particle.id)"></span></div>
      <Transition :name="viewTransition" mode="out-in" @before-leave="hideLeavingCanvas" @after-enter="focusView">
        <div v-if="activeView === 'home'" key="home" class="relative z-10 flex min-h-0 min-w-0 flex-1 flex-col justify-center overflow-y-auto lg:flex-row lg:items-center lg:overflow-visible">
          <main id="home" tabindex="-1" class="px-6 py-14 sm:pl-12 sm:pr-0 sm:py-20 lg:min-h-[32rem] lg:min-w-0 lg:basis-3/5 lg:pl-16 lg:pr-0"><div class="max-w-3xl"><p class="text-3xl font-bold tracking-[0.12em] text-violet-300 sm:text-4xl">I'm</p><h1 class="mt-1 text-5xl font-semibold tracking-[-0.055em] text-white sm:text-6xl lg:text-8xl">Irwan Cahyo Saputro</h1><p class="mt-7 max-w-xl text-base leading-7 text-slate-300 sm:text-lg sm:leading-8">I'm a front-end developer who creates smooth, visually engaging web experiences. I turn ideas into responsive interfaces that are clear, fast, and easy to use while continually learning and solving real problems.</p></div></main>
          <div class="relative hidden min-w-0 lg:flex lg:basis-2/5 lg:items-center lg:justify-center"><img src="/profile/new-profile.png" alt="Portrait of Irwan Cahyo Saputro" width="315" height="323" class="w-full max-w-[15rem] object-contain lg:max-w-[19rem]" decoding="async" /><div class="pointer-events-none absolute inset-x-0 bottom-0 z-20 h-16 bg-gradient-to-t from-[#17191e] to-transparent"></div></div>
        </div>
        <ExperiencePage v-else-if="activeView === 'experience'" key="experience" />
        <EducationPage v-else-if="activeView === 'education'" key="education" />
        <CertificationPage v-else-if="activeView === 'certification'" key="certification" />
        <SkillsPage v-else-if="activeView === 'skills'" key="skills" />
        <ProjectsPage v-else key="projects" />
      </Transition>
      <aside class="relative z-45 flex h-[56px] shrink-0 items-center justify-center gap-4 bg-transparent px-4 sm:gap-5 lg:relative lg:order-first lg:inset-auto lg:h-auto lg:w-20 lg:flex-col lg:justify-center lg:gap-8 lg:border-r lg:border-white/10 lg:px-0 lg:py-8" aria-label="Section shortcuts">
        <button type="button" :aria-pressed="activeView === 'home'" aria-label="Profile" class="rail-button" @click="navigateTo('home')"><UserRound :size="18" :stroke-width="1.6" aria-hidden="true" /><span class="sr-only">Profile</span><span class="rail-tooltip">Profile</span></button>
        <button type="button" :aria-pressed="activeView === 'experience'" aria-label="Experience" class="rail-button" @click="navigateTo('experience')"><BriefcaseBusiness :size="18" :stroke-width="1.6" aria-hidden="true" /><span class="sr-only">Experience</span><span class="rail-tooltip">Experience</span></button>
        <button type="button" :aria-pressed="activeView === 'education'" aria-label="Education" class="rail-button" @click="navigateTo('education')"><GraduationCap :size="18" :stroke-width="1.6" aria-hidden="true" /><span class="sr-only">Education</span><span class="rail-tooltip">Education</span></button>
        <button type="button" :aria-pressed="activeView === 'certification'" aria-label="Certification" class="rail-button" @click="navigateTo('certification')"><BadgeCheck :size="18" :stroke-width="1.6" aria-hidden="true" /><span class="sr-only">Certification</span><span class="rail-tooltip">Certification</span></button>
        <button type="button" :aria-pressed="activeView === 'skills'" aria-label="Skills" class="rail-button" @click="navigateTo('skills')"><Braces :size="18" :stroke-width="1.6" aria-hidden="true" /><span class="sr-only">Skills</span><span class="rail-tooltip">Skills</span></button>
        <button type="button" :aria-pressed="activeView === 'projects'" aria-label="Projects" class="rail-button" @click="navigateTo('projects')"><Code2 :size="18" :stroke-width="1.6" aria-hidden="true" /><span class="sr-only">Projects</span><span class="rail-tooltip">Projects</span></button>
      </aside>
    </div>
  </div>
</template>

<style scoped>
@property --gloss-angle { syntax: '<angle>'; inherits: false; initial-value: 125deg; }
.surface-gloss { background-image: radial-gradient(65% 65% at var(--gloss-x) var(--gloss-y), rgb(255 255 255 / 1.5%), rgb(196 181 253 / .75%) 40%, transparent 70%), linear-gradient(var(--gloss-angle), transparent 30%, rgb(255 255 255 / 1%) 45%, rgb(196 181 253 / .5%) 50%, transparent 65%); background-size: 140% 140%, 260% 260%; background-position: 0 0, var(--gloss-x) var(--gloss-y); transition: background-position 500ms ease-out, --gloss-angle 500ms ease-out; }
.view-forward-enter-active, .view-forward-leave-active, .view-back-enter-active, .view-back-leave-active { transition: opacity 280ms ease; }
.view-forward-enter-from, .view-forward-leave-to, .view-back-enter-from, .view-back-leave-to { opacity: 0; }
.view-forward-leave :deep(canvas), .view-back-leave :deep(canvas) { display: none; }
.rail-button { position: relative; display: flex; height: 2.25rem; width: 2.25rem; align-items: center; justify-content: center; border-radius: 9999px; color: #94a3b8; transition: color .2s, background .2s; }
.rail-button:hover { color: white; background: rgb(255 255 255 / .08); }
.rail-button:focus-visible { outline: 2px solid #c4b5fd; outline-offset: 2px; }
.rail-tooltip { pointer-events: none; position: absolute; left: 100%; margin-left: .75rem; display: none; white-space: nowrap; border-radius: .375rem; background: #020617; padding: .25rem .5rem; font-size: .75rem; font-weight: 500; color: white; box-shadow: 0 10px 15px rgb(0 0 0 / .2); }
@media (min-width: 1024px) { .rail-button:hover .rail-tooltip, .rail-button:focus-visible .rail-tooltip { display: block; } }
@media (prefers-reduced-motion: reduce) { .view-forward-enter-active, .view-forward-leave-active, .view-back-enter-active, .view-back-leave-active { transition-duration: 1ms; } }
.trail-bubble { width: 8px; height: 8px; margin: -4px 0 0 -4px; border-radius: 9999px; background: radial-gradient(circle at 30% 30%, #ddd6fe, #7c3aed 70%); animation: bubble-trail 650ms ease-out forwards; will-change: transform, opacity; }
@keyframes bubble-trail { from { transform: translateY(0) scale(1); opacity: .8; } to { transform: translateY(-28px) scale(.3); opacity: 0; } }
</style>
