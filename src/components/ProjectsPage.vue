<script setup lang="ts">
import { ChevronLeft, ChevronRight, ExternalLink, GitBranch } from '@lucide/vue'
import emblaCarouselVue from 'embla-carousel-vue'
import { nextTick, onUnmounted, ref, watch } from 'vue'

type Project = {
  id: string
  title: string
  stack: string
  technologies: string[]
  githubUrl?: string
  projectUrl: string
  image?: string
  tone: 'violet' | 'cyan' | 'emerald' | 'amber' | 'rose'
}

const projects: Project[] = [
  {
    id: 'optiiion',
    title: 'Optiiion',
    stack: 'Next.js',
    technologies: ['TypeScript', 'Supabase', 'Tailwind CSS', 'shadcn/ui', 'MediaPipe'],
    githubUrl: 'https://github.com/irwancahyos/simple-jobsearch',
    projectUrl: 'https://wp-jobsearch.vercel.app/',
    image: '/projects/optiiion.webp',
    tone: 'violet',
  },
  {
    id: 'sellerpintar',
    title: 'Sellerpintar',
    stack: 'Next.js',
    technologies: ['TypeScript', 'Tailwind CSS', 'Radix UI', 'Prisma', 'JWT', 'Cloudinary', 'Quill', 'Zod', 'React Hook Form', 'Vitest'],
    githubUrl: 'https://github.com/irwancahyos/sellerpintar-hometest/tree/main',
    projectUrl: 'https://sellerpintar-hometest.vercel.app/user/article',
    image: '/projects/sellerpintar-hometest.webp',
    tone: 'cyan',
  },
  {
    id: 'football-decision-making-assessment',
    title: 'Football Decision Making Assessment',
    stack: 'Next.js',
    technologies: ['TypeScript', 'Tailwind CSS', 'Motion', 'Base UI', 'React Context', 'Cloudflare Workers', 'Cloudflare D1', 'Telegram Bot API'],
    githubUrl: 'https://github.com/irwancahyos/football-assessment',
    projectUrl: 'https://fdma.pages.dev/',
    image: '/projects/football-decision-making.webp',
    tone: 'emerald',
  },
  {
    id: 'surya-pratama-lestari',
    title: 'Surya Pratama Lestari',
    stack: 'Next.js',
    technologies: ['TypeScript', 'Tailwind CSS', 'Radix UI', 'Zustand', 'Axios', 'React Hook Form', 'Tiptap', 'TanStack Table', 'Docker', 'GitHub Actions'],
    projectUrl: 'https://suryapratamalestari.com/',
    image: '/projects/surya-pratama-lestari.webp',
    tone: 'amber',
  },
]

const [emblaRef, emblaApi] = emblaCarouselVue({
  loop: true,
  align: 'center',
  skipSnaps: false,
})

type EmblaApi = NonNullable<typeof emblaApi.value>

const activeIndex = ref(0)

function updateFan(api: EmblaApi) {
  const scrollProgress = api.scrollProgress()
  const scrollSnaps = api.scrollSnapList()
  const slides = api.slideNodes()
  const viewportWidth = api.rootNode().clientWidth
  const desktop = window.matchMedia('(min-width: 1024px)').matches
  const visibleLimit = desktop ? 2.55 : 1.49
  const fanGap = desktop
    ? Math.min(viewportWidth * 0.115, 128)
    : Math.min(viewportWidth * 0.42, 156)
  const snapStep = Math.abs((scrollSnaps[1] ?? 1) - (scrollSnaps[0] ?? 0)) || 1
  const loopLength = scrollSnaps.length
  const positions = scrollSnaps.map((scrollSnap) => {
    let position = (scrollSnap - scrollProgress) / snapStep
    if (position > loopLength / 2) position -= loopLength
    if (position < -loopLength / 2) position += loopLength
    return position
  })

  const targetOffsets = slides.map((slide, slideIndex) => {
    const slideRect = slide.getBoundingClientRect()
    return api.rootNode().getBoundingClientRect().left + viewportWidth / 2
      - (slideRect.left + slideRect.width / 2)
      + positions[slideIndex] * fanGap
  })

  slides.forEach((slide, slideIndex) => {
    const position = positions[slideIndex]
    const distance = Math.abs(position)
    const visible = distance <= visibleLimit
    const card = slide.querySelector<HTMLElement>('.project-card')
    const selectButton = slide.querySelector<HTMLButtonElement>('.project-card-select')

    if (!card) return

    card.style.setProperty('--fan-x', `${targetOffsets[slideIndex]}px`)
    card.style.setProperty('--fan-y', `${Math.min(distance, 2) * 12}px`)
    card.style.setProperty('--fan-rotate', `${Math.max(-18, Math.min(18, position * 9))}deg`)
    card.style.setProperty('--fan-scale', `${1 - Math.min(distance, 2) * 0.08}`)
    card.style.setProperty('--fan-opacity', `${visible ? 1 - Math.min(distance, 2) * 0.18 : 0}`)

    slide.style.zIndex = `${100 - Math.round(distance * 10)}`
    slide.setAttribute('aria-hidden', String(!visible))
    card.style.pointerEvents = visible ? 'auto' : 'none'
    if (selectButton) selectButton.tabIndex = visible ? 0 : -1
  })
}

function syncActive(api: EmblaApi) {
  activeIndex.value = api.selectedScrollSnap()
}

async function handleSelect(api: EmblaApi) {
  syncActive(api)
  await nextTick()
  updateFan(api)
}

async function handleReInit(api: EmblaApi) {
  syncActive(api)
  await nextTick()
  updateFan(api)
}

function detachEmbla(api?: EmblaApi) {
  api
    ?.off('scroll', updateFan)
    .off('select', handleSelect)
    .off('reInit', handleReInit)
}

watch(emblaApi, (api, previousApi) => {
  void emblaRef.value
  detachEmbla(previousApi)
  if (!api) return

  syncActive(api)
  updateFan(api)
  api
    .on('scroll', updateFan)
    .on('select', handleSelect)
    .on('reInit', handleReInit)
})

onUnmounted(() => detachEmbla(emblaApi.value))

function previous() { emblaApi.value?.scrollPrev() }
function next() { emblaApi.value?.scrollNext() }
function selectProject(index: number) { emblaApi.value?.scrollTo(index) }
</script>

<template>
  <section tabindex="-1" aria-label="Projects" class="relative z-10 flex min-h-0 flex-1 overflow-hidden px-6 py-10 sm:px-12 lg:items-center lg:px-16 lg:py-0">
    <div class="flex min-h-0 w-full max-w-6xl flex-1 flex-col">
      <p class="shrink-0 text-sm font-semibold tracking-[0.14em] text-violet-300 uppercase">Projects</p>

      <div ref="emblaRef" class="embla mt-5 min-h-0 flex-1 overflow-hidden">
        <div class="embla-track flex h-full touch-pan-y">
          <div
            v-for="(project, index) in projects"
            :key="project.id"
            class="embla-slide relative min-w-0 shrink-0 basis-full"
          >
            <article
              class="project-card"
              :class="[
                `project-card-${project.tone}`,
                index === activeIndex ? 'project-card-active' : 'project-card-compact',
              ]"
            >
              <div class="project-art" :class="{ 'project-art-image': project.image }" aria-hidden="true">
                <img
                  v-if="project.image"
                  :src="project.image"
                  alt=""
                  width="640"
                  height="682"
                  loading="lazy"
                  decoding="async"
                />
              </div>

              <button
                v-if="index !== activeIndex"
                type="button"
                class="project-card-select"
                :aria-label="`Center ${project.title}`"
                @click="selectProject(index)"
              ></button>

              <div class="project-copy">
                <h1 class="project-title">{{ project.title }}</h1>
                <p class="project-stack">{{ project.stack }}</p>

                <template v-if="index === activeIndex">
                  <p class="project-technologies">{{ project.technologies.join(' · ') }}</p>
                  <div class="project-links">
                    <a
                      v-if="project.githubUrl"
                      :href="project.githubUrl"
                      target="_blank"
                      rel="noopener noreferrer"
                      class="project-link"
                      :aria-label="`Open ${project.title} source code on GitHub`"
                    >
                      <GitBranch :size="17" aria-hidden="true" />
                    </a>
                    <a
                      :href="project.projectUrl"
                      target="_blank"
                      rel="noopener noreferrer"
                      class="project-link"
                      :aria-label="`Open ${project.title} live project`"
                    >
                      <ExternalLink :size="17" aria-hidden="true" />
                    </a>
                  </div>
                </template>
              </div>
            </article>
          </div>
        </div>
      </div>

      <div class="mt-4 flex shrink-0 items-center justify-center gap-4">
        <button type="button" class="carousel-arrow" aria-label="Previous project" @click="previous">
          <ChevronLeft :size="18" />
        </button>
        <nav class="flex items-center gap-2" aria-label="Select project">
          <button
            v-for="(project, index) in projects"
            :key="project.id"
            type="button"
            class="h-1.5 rounded-full transition"
            :class="index === activeIndex ? 'w-5 bg-violet-300' : 'w-1.5 bg-white/25'"
            :aria-label="`Show ${project.title}`"
            :aria-current="index === activeIndex ? 'true' : undefined"
            @click="selectProject(index)"
          ></button>
        </nav>
        <button type="button" class="carousel-arrow" aria-label="Next project" @click="next">
          <ChevronRight :size="18" />
        </button>
      </div>
      <p class="sr-only" aria-live="polite">Current project: {{ projects[activeIndex].title }}</p>
    </div>
  </section>
</template>

<style scoped>
.embla {
  height: 430px;
  flex: 0 0 430px;
}

.embla-slide {
  pointer-events: none;
}

.project-card {
  --fan-x: 0px;
  --fan-y: 0px;
  --fan-rotate: 0deg;
  --fan-scale: 1;
  --fan-opacity: 1;
  position: absolute;
  top: 50%;
  left: 50%;
  width: min(76vw, 310px);
  aspect-ratio: 0.72;
  overflow: hidden;
  border: 1px solid rgb(255 255 255 / 0.13);
  border-radius: 1.25rem;
  background: #0d1421;
  color: #f8fafc;
  opacity: var(--fan-opacity);
  transform:
    translate3d(calc(-50% + var(--fan-x)), calc(-50% + var(--fan-y)), 0)
    rotate(var(--fan-rotate))
    scale(var(--fan-scale));
  transform-origin: 50% 115%;
  box-shadow: 0 24px 48px rgb(0 0 0 / 0.38);
  will-change: transform, opacity;
}

.project-card-active {
  border-color: rgb(196 181 253 / 0.52);
  box-shadow:
    0 30px 60px rgb(0 0 0 / 0.48),
    0 0 44px rgb(124 58 237 / 0.13);
}

.project-card-select {
  position: absolute;
  z-index: 3;
  inset: 0;
  width: 100%;
  cursor: pointer;
  border: 0;
  background: transparent;
}

.project-card-select:focus-visible {
  outline: 2px solid #c4b5fd;
  outline-offset: -4px;
}

.project-art {
  position: absolute;
  inset: 0;
  background:
    radial-gradient(circle at 72% 24%, var(--project-glow), transparent 34%),
    linear-gradient(145deg, var(--project-accent), transparent 48%);
}

.project-art img {
  height: 100%;
  width: 100%;
  object-fit: cover;
  object-position: center;
}

.project-art-image {
  background: #f8f8f8;
}

.project-art::after {
  position: absolute;
  inset: 0;
  background: linear-gradient(to top, rgb(5 9 17 / 0.97) 8%, rgb(5 9 17 / 0.72) 42%, transparent 76%);
  content: '';
}

.project-card-compact .project-art-image::after {
  background: linear-gradient(to top, rgb(5 9 17 / 0.96) 10%, rgb(5 9 17 / 0.56) 58%, rgb(5 9 17 / 0.14));
}

.project-card-violet { --project-accent: rgb(109 40 217 / 0.58); --project-glow: rgb(217 70 239 / 0.32); }
.project-card-cyan { --project-accent: rgb(8 145 178 / 0.5); --project-glow: rgb(59 130 246 / 0.3); }
.project-card-emerald { --project-accent: rgb(5 150 105 / 0.48); --project-glow: rgb(34 211 238 / 0.25); }
.project-card-amber { --project-accent: rgb(180 83 9 / 0.5); --project-glow: rgb(239 68 68 / 0.24); }
.project-card-rose { --project-accent: rgb(190 24 93 / 0.48); --project-glow: rgb(139 92 246 / 0.28); }

.project-copy {
  position: absolute;
  z-index: 2;
  right: 1.25rem;
  bottom: 1.25rem;
  left: 1.25rem;
}

.project-title {
  font-size: 1.35rem;
  font-weight: 650;
  line-height: 1.15;
}

.project-stack {
  margin-top: 0.55rem;
  font-size: 0.75rem;
  font-weight: 600;
  color: #c4b5fd;
}

.project-technologies {
  margin-top: 0.8rem;
  font-size: 0.7rem;
  line-height: 1.5;
  color: #cbd5e1;
}

.project-links {
  position: relative;
  z-index: 4;
  margin-top: 1rem;
  display: flex;
  gap: 0.6rem;
}

.project-link {
  display: inline-flex;
  height: 2.25rem;
  width: 2.25rem;
  align-items: center;
  justify-content: center;
  border: 1px solid rgb(255 255 255 / 0.16);
  border-radius: 9999px;
  color: #e2e8f0;
  transition: border-color 0.2s, background 0.2s, color 0.2s;
}

.project-link:hover {
  border-color: rgb(196 181 253 / 0.5);
  background: rgb(255 255 255 / 0.08);
  color: white;
}

.project-link:focus-visible {
  outline: 2px solid #c4b5fd;
  outline-offset: 2px;
}

.project-card-compact .project-copy {
  right: 1rem;
  bottom: 1rem;
  left: 1rem;
}

.project-card-compact .project-title {
  font-size: 1rem;
}

.carousel-arrow {
  display: flex;
  height: 2rem;
  width: 2rem;
  align-items: center;
  justify-content: center;
  border-radius: 9999px;
  color: #94a3b8;
  transition: background 0.2s, color 0.2s;
}

.carousel-arrow:hover {
  background: rgb(255 255 255 / 0.08);
  color: white;
}

@media (min-width: 1024px) {
  .embla {
    height: 480px;
    flex-basis: 480px;
  }

  .project-card {
    width: clamp(230px, 23vw, 286px);
  }
}

@media (prefers-reduced-motion: reduce) {
  .project-card,
  .carousel-arrow,
  nav button {
    transition: none;
  }
}
</style>
