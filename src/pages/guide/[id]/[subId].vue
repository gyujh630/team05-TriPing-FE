<template>
  <div v-if="!guideLoaded" class="p-6 max-w-3xl mx-auto space-y-4">
    <div class="text-center text-xl text-gray-500 my-8">AI 가이드북을 생성하고 있습니다... 🤖</div>
    <Skeleton height="2rem" width="70%" />
    <Skeleton height="1.5rem" width="90%" />
    <Skeleton height="10rem" />
    <Skeleton height="2rem" width="60%" />
    <Skeleton height="1.5rem" width="80%" />
    <Skeleton height="10rem" />
  </div>
  <transition name="fade-up" appear>
      <div v-if="guideLoaded" class="p-6 max-w-3xl mx-auto space-y-8 pb-32">
        <h1 class="md:text-3xl text-2xl font-bold text-gray-800 flex items-center gap-2">
      <i
        class="pi pi-angle-left text-gray-600 text-3xl cursor-pointer pr-2"
        @click="$router.go(-1)"
      ></i>
      {{ guide.title }} - {{ guide.subtitle }} 가이드
    </h1>
        <!-- 탭 버튼 -->
        <div class="sticky top-0 z-40 pt-4 pb-6 blur-gradient-header" v-if="tabs.length">
          <div class="relative">
            <button
              @click="scrollLeft"
              class="hidden md:flex items-center justify-center absolute left-0 top-1/2 -translate-y-1/2 z-10 bg-white rounded-full shadow w-8 h-8 cursor-pointer"
            >
              <i class="pi pi-angle-left text-gray-600"></i>
            </button>

            <div
              ref="tabScrollRef"
              class="flex gap-2 overflow-x-auto whitespace-nowrap scrollbar-none px-1 -mx-1 md:pl-10 md:pr-10"
            >
              <button
                v-for="tab in tabs"
                :key="tab.key"
                @click="scrollToSection(tab.key)"
                :class="[
                  'px-4 py-2 rounded-3xl border text-sm font-medium transition shrink-0 cursor-pointer',
                  selectedTab === tab.key
                    ? 'bg-primary text-white border-primary'
                    : 'bg-white text-gray-800 border-gray-300 hover:bg-gray-100'
                ]"
              >
                {{ tab.label }}
              </button>
            </div>

            <button
              @click="scrollRight"
              class="hidden md:flex items-center justify-center absolute right-0 top-1/2 -translate-y-1/2 z-10 bg-white rounded-full shadow w-8 h-8 cursor-pointer"
            >
              <i class="pi pi-angle-right text-gray-600"></i>
            </button>
          </div>
        </div>

        <!-- 상세 정보 -->
        <Card
          v-if="hasInfo"
          id="info"
          class="scroll-mt-24 md:shadow-sm md:border-gray-100 md:border-1 border-0 shadow-none"
        >
          <template #title>🗽️ 상세 정보</template>
          <template #content>
            <p class="text-base text-gray-700 leading-relaxed whitespace-pre-line p-2">
              {{ guide.info }}
            </p>
          </template>
        </Card>

        <!-- 꿀팁 -->
        <Card
          v-if="hasTip"
          id="tip"
          class="scroll-mt-24 md:shadow-sm md:border-gray-100 md:border-1 border-0 shadow-none"
        >
          <template #title>✨ 꿀팁</template>
          <template #content>
            <ul class="list-disc pl-6 text-base text-gray-700 leading-relaxed space-y-2">
              <li v-for="(item, index) in guide.tip" :key="index">{{ item }}</li>
            </ul>
          </template>
        </Card>

        <!-- 역사 -->
        <Card
          v-if="hasHistory"
          id="history"
          class="scroll-mt-24 md:shadow-sm md:border-gray-100 md:border-1 border-0 shadow-none"
        >
          <template #title>📚 역사</template>
          <template #content>
            <p class="text-base text-gray-700 leading-relaxed whitespace-pre-line p-2">
              {{ guide.history }}
            </p>
          </template>
        </Card>
        
      </div>
  </transition>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'
import { useRoute } from 'vue-router'
import Card from 'primevue/card'
import { useVisitedSubGuideStore } from '@/stores/visitedSubGuideStore'

const visitedSubStore = useVisitedSubGuideStore()

const route = useRoute()
const title = computed(() => route.query.title || '')
const subtitle = computed(() => route.query.subtitle || '')

const guide = ref(null)
const guideLoaded = computed(() => guide.value !== null)

const hasInfo = computed(() => !!guide.value?.info)
const hasTip = computed(() => guide.value?.tip?.length > 0)
const hasHistory = computed(() => !!guide.value?.history)

const tabs = computed(() => {
  const result = []
  if (hasInfo.value) result.push({ label: '상세 정보', key: 'info' })
  if (hasTip.value) result.push({ label: '꿀팁', key: 'tip' })
  if (hasHistory.value) result.push({ label: '역사', key: 'history' })
  return result
})

const selectedTab = ref('info')
const tabScrollRef = ref(null)

function scrollToSection(id) {
  const el = document.getElementById(id)
  if (el) el.scrollIntoView({ behavior: 'smooth' })
  selectedTab.value = id
}

function scrollLeft() {
  tabScrollRef.value?.scrollBy({ left: -150, behavior: 'smooth' })
}

function scrollRight() {
  tabScrollRef.value?.scrollBy({ left: 150, behavior: 'smooth' })
}

async function fetchSubGuide(title, subtitle) {
  try {
    const start = Date.now()

    const res = await fetch(
      `https://tri-ping.site/attraction/subguide?title=${encodeURIComponent(title)}&subTitle=${encodeURIComponent(subtitle)}`
    )
    if (!res.ok) throw new Error('서브 가이드 API 호출 실패')
    const data = await res.json()

    const elapsed = Date.now() - start
    const alreadyVisited = visitedSubStore.has(title, subtitle)
    visitedSubStore.add(title, subtitle)

    const wait = !alreadyVisited && elapsed < 2000 ? 2000 - elapsed : 0
    await new Promise((resolve) => setTimeout(resolve, wait))

    guide.value = data
  } catch (err) {
    console.error('[SubGuidePage] API 오류:', err)
    guide.value = null
  }
}

onMounted(() => {
  if (title.value && subtitle.value) {
    fetchSubGuide(title.value, subtitle.value)
  } else {
    console.warn('[SubGuidePage] 쿼리 파라미터 누락')
    guide.value = null
  }
})
</script>