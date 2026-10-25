<script setup lang="ts">
import { OrbitControls, useTexture } from '@tresjs/cientos'
import { TresCanvas } from '@tresjs/core'
import { computed, onMounted, onUnmounted, ref } from 'vue'

const prefersReducedMotion = ref(false)
let motionQuery: MediaQueryList | undefined

function updateMotionPreference() {
  prefersReducedMotion.value = motionQuery?.matches ?? false
}

onMounted(() => {
  motionQuery = window.matchMedia('(prefers-reduced-motion: reduce)')
  updateMotionPreference()
  motionQuery.addEventListener('change', updateMotionPreference)
})

onUnmounted(() => {
  motionQuery?.removeEventListener('change', updateMotionPreference)
})

const shouldAutoRotate = computed(() => !prefersReducedMotion.value)
const { state: globeTexture, isLoading: isTextureLoading } = useTexture('/education/globe.jpg')
</script>

<template>
  <section tabindex="-1" aria-label="Education" class="relative z-10 min-h-0 flex-1 overflow-y-auto px-6 py-10 sm:px-12 lg:flex lg:items-center lg:overflow-visible lg:px-16 lg:py-0">
    <img
      src="/education/graduation-new.png"
      alt=""
      aria-hidden="true"
      class="pointer-events-none absolute inset-x-0 bottom-0 h-auto w-full opacity-15"
    />

    <div class="relative z-10 w-full max-w-6xl lg:pr-[44%]">
      <div>
        <p class="text-sm font-semibold tracking-[0.14em] text-violet-300 uppercase">Education</p>
        <h1 class="mt-5 text-4xl font-semibold tracking-[-0.055em] text-white sm:text-5xl lg:text-6xl">Yogyakarta State University</h1>
        <p class="mt-3 text-lg text-slate-300">Bachelor of Sport Science</p>
        <p class="mt-1 text-sm font-semibold tracking-[0.14em] text-slate-500 uppercase">2016 — 2020 · GPA 3.49</p>
        <ul class="mt-8 max-w-xl space-y-3 text-base leading-7 text-slate-300">
          <li>Studied professional football coaching science, athlete anatomy, and performance-focused training methods.</li>
          <li>Assisted national coaching license events by analyzing training programs and serving as a demonstration model.</li>
          <li>Completed a thesis on portable wheels for easy transfer of football goals.</li>
        </ul>
      </div>

      <div class="pointer-events-none absolute inset-0 z-0 h-full w-full opacity-25 lg:pointer-events-auto lg:absolute lg:inset-auto lg:right-0 lg:top-1/2 lg:z-20 lg:h-[560px] lg:w-[42vw] lg:max-w-[680px] lg:-translate-y-1/2 lg:cursor-grab lg:overflow-visible lg:opacity-100 lg:active:cursor-grabbing">
        <TresCanvas :dpr="[1, 2]" :alpha="true" :clear-alpha="0">
          <TresPerspectiveCamera :position="[3.8, 2.8, 5.5]" :fov="42" :look-at="[0, 0, 0]" />
          <OrbitControls :enable-damping="true" :damping-factor="0.08" :enable-pan="false" :enable-zoom="false" :auto-rotate="shouldAutoRotate" :auto-rotate-speed="0.8" />
          <TresAmbientLight :intensity="1.4" />
          <TresDirectionalLight :position="[4, 6, 3]" :intensity="2.2" color="#c4b5fd" />
          <TresPointLight :position="[-3, 1, -2]" :intensity="6" color="#7c3aed" />

          <TresGroup :rotation="[0.2, -0.4, 0]">
            <TresMesh>
              <TresSphereGeometry :args="[1.3, 48, 48]" />
              <TresMeshStandardMaterial
                :map="isTextureLoading ? null : globeTexture"
                :emissive-map="isTextureLoading ? null : globeTexture"
                emissive="#ffffff"
                :emissive-intensity="0.4"
                color="#ffffff"
                :metalness="0.1"
                :roughness="0.55"
              />
            </TresMesh>
            <TresGroup :rotation="[0.35, 0.5, 0.05]">
              <TresMesh>
                <TresTorusGeometry :args="[1.85, 0.012, 8, 64]" />
                <TresMeshBasicMaterial color="#c4b5fd" transparent :opacity="0.15" />
              </TresMesh>
              <TresGroup :position="[-0.633, 1.738, 0]" :rotation="[0.15, 0.55, 0]">
                <TresMesh :rotation="[0, 0.78, 0]">
                  <TresBoxGeometry :args="[0.5, 0.04, 0.5]" />
                  <TresMeshStandardMaterial color="#312e81" :metalness="0.55" :roughness="0.25" />
                </TresMesh>
                <TresMesh :position="[0, -0.14, 0]">
                  <TresCylinderGeometry :args="[0.14, 0.16, 0.24, 16]" />
                  <TresMeshStandardMaterial color="#fbbf24" :metalness="0.75" :roughness="0.2" />
                </TresMesh>
              </TresGroup>
            </TresGroup>

            <TresGroup :rotation="[-0.25, -0.35, 0.15]">
              <TresMesh>
                <TresTorusGeometry :args="[2.05, 0.012, 8, 64]" />
                <TresMeshBasicMaterial color="#7c3aed" transparent :opacity="0.15" />
              </TresMesh>
              <TresMesh :position="[1.858, 0.866, 0]" :rotation="[0.4, 0.5, 0]">
                <TresBoxGeometry :args="[0.3, 0.3, 0.3]" />
                <TresMeshBasicMaterial color="#c4b5fd" wireframe />
              </TresMesh>
            </TresGroup>

            <TresGroup :rotation="[0.5, -0.6, -0.1]">
              <TresMesh>
                <TresTorusGeometry :args="[2.25, 0.012, 8, 64]" />
                <TresMeshBasicMaterial color="#fbbf24" transparent :opacity="0.15" />
              </TresMesh>
              <TresMesh :position="[-0.77, -2.114, 0]">
                <TresSphereGeometry :args="[0.24, 20, 20]" />
                <TresMeshStandardMaterial color="#f8fafc" :metalness="0.2" :roughness="0.45" />
              </TresMesh>
            </TresGroup>

            <TresMesh :position="[2.48, 1.4, -0.62]">
              <TresSphereGeometry :args="[0.05, 12, 12]" />
              <TresMeshBasicMaterial color="#c4b5fd" />
            </TresMesh>
            <TresMesh :position="[-2.33, -0.93, 1.24]">
              <TresSphereGeometry :args="[0.04, 12, 12]" />
              <TresMeshBasicMaterial color="#7c3aed" />
            </TresMesh>
            <TresMesh :position="[1.4, -1.71, -1.55]">
              <TresSphereGeometry :args="[0.06, 12, 12]" />
              <TresMeshBasicMaterial color="#fbbf24" />
            </TresMesh>
            <TresMesh :position="[-1.47, 2.02, 0.47]">
              <TresSphereGeometry :args="[0.045, 12, 12]" />
              <TresMeshBasicMaterial color="#f8fafc" />
            </TresMesh>
            <TresMesh :position="[2.02, -0.31, 1.86]">
              <TresSphereGeometry :args="[0.05, 12, 12]" />
              <TresMeshBasicMaterial color="#c4b5fd" />
            </TresMesh>
            <TresMesh :position="[-1.86, 0.16, -1.71]">
              <TresSphereGeometry :args="[0.04, 12, 12]" />
              <TresMeshBasicMaterial color="#7c3aed" />
            </TresMesh>
            <TresMesh :position="[0.47, 2.33, -1.09]">
              <TresSphereGeometry :args="[0.05, 12, 12]" />
              <TresMeshBasicMaterial color="#fbbf24" />
            </TresMesh>
          </TresGroup>
        </TresCanvas>
        <p class="pointer-events-none absolute bottom-4 left-1/2 hidden -translate-x-1/2 whitespace-nowrap text-xs font-semibold tracking-[0.14em] text-slate-500 uppercase lg:block">Drag to explore</p>
      </div>
    </div>
  </section>
</template>
