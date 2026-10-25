<script setup lang="ts">
import { Award, BadgeCheck, Code2, Gauge, Palette } from '@lucide/vue'

const certifications = [
  {
    id: 'zettachamp',
    title: 'ZettaChamp',
    issuer: 'Zettabyte',
    year: 'Credential',
    description: 'Angular and GraphQL Client bootcamp.',
    url: 'https://drive.google.com/file/d/11RFfwFTvj4PO9Dbv3LWYbeWXU2gjAgo7/view?usp=sharing',
    icon: Award,
    tone: 'violet',
  },
  {
    id: 'javascript-algorithms',
    title: 'JavaScript Algorithms',
    issuer: 'FreeCodeCamp',
    year: 'Credential',
    description: 'Algorithms and data structures in JavaScript.',
    url: 'https://freecodecamp.org/certification/irwancahyos/javascript-algorithms-and-data-structures',
    icon: Code2,
    tone: 'teal',
  },
  {
    id: 'react-frontend',
    title: 'React Js Web Front End',
    issuer: 'Bootcamp',
    year: 'Credential',
    description: 'React, state management, APIs, and deployment.',
    url: 'https://drive.google.com/file/d/1hkPxL9DTn73CXPDtzQ0Hg7Aw_tZjVvzz/view',
    icon: BadgeCheck,
    tone: 'amber',
  },
  {
    id: 'responsive-web-design',
    title: 'Responsive Web Design',
    issuer: 'FreeCodeCamp',
    year: 'Credential',
    description: 'Responsive interfaces and practical user cases.',
    url: 'https://www.freecodecamp.org/certification/irwancahyos/responsive-web-design',
    icon: Palette,
    tone: 'magenta',
  },
  {
    id: 'web-design-programming',
    title: 'Web Design & Programming',
    issuer: 'Bootcamp',
    year: 'Credential',
    description: 'Web fundamentals, JavaScript, CSS, HTML, and PHP.',
    url: 'https://drive.google.com/file/d/1OhnRiDCy0ZQGqWI0_702YxNalDc4w0xU/view',
    icon: Gauge,
    tone: 'blue',
  },
]
</script>

<template>
  <section tabindex="-1" aria-label="Certification" class="relative z-10 flex min-h-0 flex-1 flex-col overflow-hidden px-6 pt-10 pb-0 sm:px-12 lg:overflow-visible lg:px-16 lg:pt-20 lg:pb-0">
    <div class="mx-auto flex min-h-0 w-full max-w-4xl flex-1 flex-col">
      <h1 class="text-sm font-semibold tracking-[0.14em] text-violet-300 uppercase">Certification</h1>

      <div class="cards-scroll mt-8 min-h-0 max-h-[calc(100svh-84px)] overflow-y-auto lg:mt-16 lg:max-h-none lg:overflow-visible">
        <div class="certification-grid grid grid-cols-1 gap-3 sm:gap-4 lg:relative lg:mx-auto lg:block lg:h-[440px] lg:w-[640px]">
          <a
            v-for="certificate in certifications"
            :key="certificate.id"
            :href="certificate.url"
            target="_blank"
            rel="noopener noreferrer"
            :aria-label="`Open ${certificate.title} credential`"
            class="certification-card group flex aspect-square flex-col items-center justify-center rounded-2xl border border-white/10 p-5 text-center outline-none transition duration-300 hover:border-white/25 hover:shadow-lg hover:shadow-black/25 focus-visible:outline-2 focus-visible:outline-violet-300 lg:absolute lg:h-48 lg:w-48 lg:p-5"
            :class="`tone-${certificate.tone}`"
          >
            <div class="mx-auto flex h-16 w-16 items-center justify-center rounded-2xl bg-white/10 text-white transition group-hover:bg-white/20 lg:h-9 lg:w-9">
              <component :is="certificate.icon" :size="30" :stroke-width="1.6" aria-hidden="true" />
            </div>
            <h2 class="mt-6 text-2xl font-semibold text-white lg:mt-3 lg:text-sm">{{ certificate.title }}</h2>
            <p class="mt-3 text-base text-white/75 lg:mt-1 lg:text-[10px]">{{ certificate.issuer }}</p>
            <p class="mt-6 text-sm font-semibold tracking-[0.14em] text-white/65 uppercase lg:mt-3 lg:text-[10px]">{{ certificate.year }}</p>
          </a>
        </div>
      </div>
    </div>
  </section>
</template>

<style scoped>
.tone-violet { background: linear-gradient(145deg, #332071, #6b3dd4); }
.tone-teal { background: linear-gradient(145deg, #123d43, #19696a); }
.tone-amber { background: linear-gradient(145deg, #4d261a, #8a4d29); }
.tone-magenta { background: linear-gradient(145deg, #622052, #b5478a); }
.tone-blue { background: linear-gradient(145deg, #182a60, #2763a6); }

@media (min-width: 1024px) {
  .certification-card:nth-child(1) { top: 0; left: 112px; transform: rotate(-3deg); z-index: 3; }
  .certification-card:nth-child(2) { top: 8px; left: 352px; transform: rotate(3deg); z-index: 2; }
  .certification-card:nth-child(3) { top: 194px; left: 0; transform: rotate(-2deg); z-index: 4; }
  .certification-card:nth-child(4) { top: 202px; left: 216px; transform: rotate(1deg); z-index: 5; }
  .certification-card:nth-child(5) { top: 198px; left: 432px; transform: rotate(4deg); z-index: 1; }
  .certification-card:hover { transform: translateY(-10px) rotate(0deg); z-index: 10; }
}

@media (prefers-reduced-motion: reduce) {
  .certification-card { transition: none; transform: none !important; }
}
</style>
