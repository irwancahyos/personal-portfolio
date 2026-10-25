<script setup lang="ts">
import { ArrowLeft, ArrowRight } from '@lucide/vue'
import { computed, ref } from 'vue'

const experiences = [
  {
    id: 'dealls',
    title: 'Front End Developer',
    company: 'Dealls',
    period: 'Dec 2025 — Present',
    technologies: ['Jira', 'Next.js', 'React.js', 'Astro', 'TanStack Query', 'Postman'],
    responsibilities: [
      'Architected and optimized high-traffic Dealls.com modules with React.js and Next.js for thousands of concurrent users.',
      'Worked with FE/BE leads, managers, PMs, and UI/UX designers to turn complex requirements into scalable frontend features.',
      'Aligned Backend, QA, and Mobile teams for consistent API integration and high-fidelity experiences across platforms.',
      'Implemented real-time features and high-performance components while maintaining code quality and architectural consistency.',
    ],
  },
  {
    id: 'awbs-network',
    title: 'Front End Developer',
    company: 'AWBS Network',
    period: 'Dec 2025 — Feb 2026',
    technologies: ['Next.js', 'TypeScript', 'Tailwind CSS', 'Bootstrap', 'WebSockets'],
    responsibilities: [
      'Architected a high-performance internal AI platform with Next.js, TypeScript, Tailwind CSS, and Bootstrap.',
      'Engineered end-to-end features with WebSockets for real-time processing and AI-driven feedback loops.',
      'Collaborated with Backend, QA, System Analysts, and AI teams to integrate proprietary AI models into the frontend.',
      'Managed delivery through Jira while maintaining performance and code quality for confidential enterprise solutions.',
    ],
  },
  {
    id: 'sellerpintar',
    title: 'Front End Developer',
    company: 'PT Sellerpintar Digital Asia',
    period: 'May 2025 — Oct 2025',
    technologies: ['Next.js', 'JavaScript', 'TypeScript', 'Tailwind CSS', 'SWR', 'Zustand', 'Zod'],
    responsibilities: [
      'Developed a customizable online-store platform with a real-time editor for colors, text, layout, and visual elements.',
      'Built frontend applications with Next.js, JavaScript, TypeScript, and Tailwind CSS.',
      'Integrated REST APIs with SWR for efficient data fetching and caching.',
      'Implemented state management with React Context and Zustand, plus forms with React Hook Form and Zod.',
      'Focused on scalable, reusable, and maintainable UI components.',
    ],
  },
  {
    id: 'zettabyte',
    title: 'Front End Developer',
    company: 'PT Zettabyte Pte Ltd',
    period: 'Jul 2023 — Jan 2025',
    technologies: ['Jira', 'Playwright', 'Cypress', 'Angular', 'Angular Material', 'SCSS', 'Bootstrap', 'RxJS', 'GraphQL'],
    responsibilities: [
      'Developed responsive Angular UI following internal design guidelines to improve user experience.',
      'Proposed and implemented UI/UX improvements adopted into the production platform.',
      'Integrated GraphQL APIs to deliver real-time data based on user needs.',
    ],
  },
]

const selectedExperienceId = ref<string | null>(null)
const pendingExperienceId = ref<string | null>(null)
const isRoleTransitioning = ref(false)
const roleTransitionDirection = ref<'next' | 'previous'>('next')
const touchStart = ref({ x: 0, y: 0 })
const selectedExperience = computed(() => experiences.find((experience) => experience.id === selectedExperienceId.value))
const currentExperienceIndex = computed(() => selectedExperienceId.value ? experiences.findIndex((experience) => experience.id === selectedExperienceId.value) : -1)
const hasNextExperience = computed(() => currentExperienceIndex.value < experiences.length - 1)
const hasPreviousExperience = computed(() => currentExperienceIndex.value > 0)

function selectExperience(id: string) {
  if (id === selectedExperienceId.value || isRoleTransitioning.value) return
  pendingExperienceId.value = id
  isRoleTransitioning.value = true
}

function goToNextExperience() {
  if (!hasNextExperience.value) return
  roleTransitionDirection.value = 'next'
  selectExperience(experiences[currentExperienceIndex.value + 1].id)
}

function goToPreviousExperience() {
  if (!hasPreviousExperience.value) return
  roleTransitionDirection.value = 'previous'
  selectExperience(experiences[currentExperienceIndex.value - 1].id)
}

function startSwipe(event: PointerEvent) {
  touchStart.value = { x: event.clientX, y: event.clientY }
}

function endSwipe(event: PointerEvent) {
  const deltaX = event.clientX - touchStart.value.x
  const deltaY = event.clientY - touchStart.value.y
  if (Math.abs(deltaX) < 50 || Math.abs(deltaX) <= Math.abs(deltaY)) return
  if (deltaX < 0) goToNextExperience()
  else goToPreviousExperience()
}

function commitExperience() {
  selectedExperienceId.value = pendingExperienceId.value
  pendingExperienceId.value = null
  isRoleTransitioning.value = false
}
</script>

<template>
  <section
    id="experience"
    tabindex="-1"
    aria-label="Experience"
    @pointerdown="startSwipe"
    @pointerup="endSwipe"
    class="relative z-10 flex touch-pan-y min-h-0 flex-1 flex-col items-center justify-center overflow-hidden text-center sm:px-8 sm:py-12 lg:h-auto lg:overflow-hidden lg:px-16 lg:py-0"
    :class="{ 'items-start justify-start': selectedExperience }"
  >
    <h1 v-if="!selectedExperience" class="experience-heading relative z-20 text-5xl font-semibold tracking-[-0.055em] text-white sm:text-6xl lg:text-8xl">Experience</h1>

    <div v-if="isRoleTransitioning" class="role-transition-overlay absolute inset-0 z-35 pointer-events-none" :class="{ 'from-left': roleTransitionDirection === 'previous' }" aria-hidden="true" @animationend="commitExperience"></div>

    <Transition name="sheet-reveal">
      <article v-if="selectedExperience" :key="selectedExperience.id" tabindex="-1" aria-live="polite" class="page-sheet static z-30 flex h-full min-h-0 flex-col justify-start bg-[#17191e] px-4 pt-8 text-left sm:px-12 sm:py-20 lg:absolute lg:inset-0 lg:h-auto lg:justify-center lg:px-16">
        <div class="role-content flex h-full min-h-0 max-w-3xl flex-col lg:block lg:h-auto">
          <div class="shrink-0">
            <p class="text-sm font-semibold tracking-[0.14em] text-violet-300 uppercase">Experience</p>
            <p class="mt-5 text-sm font-semibold tracking-[0.14em] text-slate-400 uppercase">{{ selectedExperience.company }} · {{ selectedExperience.period }}</p>
            <h2 class="mt-4 text-4xl font-semibold tracking-[-0.055em] text-white sm:text-5xl lg:text-7xl">{{ selectedExperience.title }}</h2>
            <p class="mt-7 text-sm text-slate-400"><span class="mr-3 font-semibold tracking-[0.14em] text-slate-500 uppercase">Stack</span>{{ selectedExperience.technologies.join(' · ') }}</p>
            <p class="mt-7 text-sm font-semibold tracking-[0.14em] text-slate-400 uppercase">What I built</p>
          </div>
          <ul class="mt-3 min-h-0 max-w-2xl flex-1 list-disc space-y-2 overflow-y-auto pb-0 pl-5 text-base leading-7 text-slate-300 sm:text-lg sm:leading-8 lg:flex-none lg:overflow-visible lg:pb-0">
            <li v-for="responsibility in selectedExperience.responsibilities" :key="responsibility">{{ responsibility }}</li>
          </ul>
        </div>
      </article>
    </Transition>

    <div v-if="hasNextExperience || hasPreviousExperience" class="swipe-cue pointer-events-none fixed top-1/2 z-40 -translate-y-1/2 lg:hidden" :class="hasNextExperience ? 'right-0' : 'left-0'" aria-hidden="true">
      <ArrowRight v-if="hasNextExperience" :size="20" :stroke-width="1.5" class="arrow-cue pulse-right text-violet-300" />
      <ArrowLeft v-else :size="20" :stroke-width="1.5" class="arrow-cue pulse-left text-violet-300" />
    </div>

    <div class="role-selector absolute bottom-6 left-6 z-40 hidden max-w-[calc(100%-3rem)] flex-wrap gap-2 text-left lg:flex">
      <button v-for="experience in experiences" :key="experience.id" type="button" class="max-w-[12rem] truncate rounded-full border border-white/15 bg-[#17191e]/90 px-4 py-2 text-sm text-slate-300 shadow-lg transition hover:bg-white/8 hover:text-white focus-visible:outline-2 focus-visible:outline-violet-300" :class="experience.id === selectedExperienceId ? 'border-violet-400/50 bg-violet-500/20 text-white' : ''" :aria-pressed="experience.id === selectedExperienceId" :title="experience.company" @click="selectExperience(experience.id)">{{ experience.company }}</button>
    </div>
  </section>
</template>

<style scoped>
.experience-heading { padding-right: 0.08em; margin-right: -0.08em; animation: experience-heading-reveal 520ms ease-out both; }
@keyframes experience-heading-reveal { from { clip-path: inset(0 100% 0 0); opacity: 0; } to { clip-path: inset(0 0 0 0); opacity: 1; } }
.sheet-reveal-leave-active { z-index: 20; transition: opacity 1ms; }
.sheet-reveal-enter-active { z-index: 30; }
.role-transition-overlay { animation: corner-wrinkle 680ms ease-out both; }
.role-transition-overlay::after { position: absolute; top: 0; right: 0; width: 100%; height: 100%; max-width: 20rem; max-height: 20rem; content: ''; border-top: 1px solid rgb(255 255 255 / 16%); border-right: 1px solid rgb(255 255 255 / 12%); border-radius: 0 0 0 100%; background: radial-gradient(circle at 100% 0, rgb(196 181 253 / 12%), transparent 68%); box-shadow: -16px 16px 28px rgb(0 0 0 / 38%); }
.role-transition-overlay.from-left::after { right: auto; left: 0; border-right: none; border-left: 1px solid rgb(255 255 255 / 12%); border-radius: 0 0 100% 0; background: radial-gradient(circle at 0 0, rgb(196 181 253 / 12%), transparent 68%); box-shadow: 16px 16px 28px rgb(0 0 0 / 38%); }
@media (max-width: 1023px) { .role-transition-overlay::after { width: 14rem; height: 14rem; max-width: 14rem; max-height: 14rem; } }
.role-content { animation: role-content-reveal 420ms ease-out both; }
@keyframes corner-wrinkle { from { opacity: 0; } 35% { opacity: 1; } to { opacity: 0; } }
@keyframes role-content-reveal { from { clip-path: polygon(100% 0, 100% 0, 100% 0, 100% 0); opacity: 0; } to { clip-path: polygon(0 0, 100% 0, 100% 100%, 0 100%); opacity: 1; } }
.arrow-cue { opacity: 0.3; }
.arrow-cue.pulse-right { animation: arrow-cue-right 2.4s ease-in-out infinite; }
.arrow-cue.pulse-left { animation: arrow-cue-left 2.4s ease-in-out infinite; }
@keyframes arrow-cue-right { 0%,100% { opacity: .3; transform: translateX(0); } 50% { opacity: .6; transform: translateX(5px); } }
@keyframes arrow-cue-left { 0%,100% { opacity: .3; transform: translateX(0); } 50% { opacity: .6; transform: translateX(-5px); } }
@media (prefers-reduced-motion: reduce) { .arrow-cue, .experience-heading, .sheet-reveal-leave-active, .role-content, .role-transition-overlay { animation-duration: 1ms; animation-delay: 0ms; transition-duration: 1ms; } .arrow-cue { opacity: .5; transform: none; } }
</style>
