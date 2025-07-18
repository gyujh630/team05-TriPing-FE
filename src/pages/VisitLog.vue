<template>
  <div class="p-4">
    <!-- 날짜 이동 버튼과 제목 -->
    <div class="flex items-center justify-between mb-6">
      <!-- 이전 버튼 -->
      <Button
        icon="pi pi-arrow-left"
        class="btn-primary text-sm md:w-32 w-16"
        :class="{ 'opacity-0 pointer-events-none': page <= 1 }"
        @click="goToPrevPage"
      >
        <span class="hidden md:inline">← 이전 날짜로</span>
        <span class="inline md:hidden">이전</span>
      </Button>

      <!-- 날짜 -->
      <div class="text-xl text-center font-semibold">
        {{ dayjs(logs[0]?.enteredAt).format('YYYY년 M월 D일') }}
      </div>

      <!-- 다음 버튼 -->
      <Button
        icon="pi pi-arrow-right"
        iconPos="right"
        class="btn-primary text-sm md:w-32 w-16"
        :class="{ 'opacity-0 pointer-events-none': page >= totalCount }"
        @click="goToNextPage"
      >
        <span class="hidden md:inline">다음 날짜로 →</span>
        <span class="inline md:hidden">다음</span>
      </Button>
    </div>

    <VisitMap :logs="logs" />
    <div v-if="visitSummary" class="text-center my-6 px-4">
  <p class="text-base md:text-lg font-medium text-gray-700">
    {{ visitSummary }}
  </p>
</div>

    <VisitLogList :logs="logs" @preference-click="handlePreferenceClick" />
    <VisitPrefModal
  :visible="visitPrefVisible"
  :visit-log-id="visitPrefId"
  :title="visitPrefTitle"
  :initial-score="visitPrefScore"
  @update:visible="visitPrefVisible = $event"
  @submit="handlePrefSubmit"
/>

    <!-- 하단 페이지네이션 -->
    <div class="flex justify-center mt-10 md:mb-0 mb-12">
    <!-- 데스크탑용 풀 페이지네이션 -->
    <Paginator 
      :rows="1" 
      :totalRecords="totalCount" 
      :first="page - 1" 
      @page="onPageChange"
      class="hidden md:block"
    />
    
    <!-- 모바일용 간단 네비게이션 -->
      <div class="flex items-center gap-4 md:hidden">
    <Button
      icon="pi pi-chevron-left"
      class="border-0 text-primary bg-transparent hover:none hover:text-primary hover:bg-transparent focus:border-primary focus:text-primary focus:bg-transparent active:border-primary active:text-primary active:bg-transparent transition-none p-2"
      :disabled="page <= 1"
      @click="goToPrevPage"
    />
    
    <span class="text-sm font-medium">
      {{ page }} / {{ totalCount }}
    </span>
    
    <Button
      icon="pi pi-chevron-right"
      class="border-0 text-primary bg-transparent hover:border-primary hover:text-primary hover:bg-transparent focus:border-primary focus:text-primary focus:bg-transparent active:border-primary active:text-primary active:bg-transparent transition-none p-2"
      :disabled="page >= totalCount"
      @click="goToNextPage"
    />
  </div>
  </div>
  </div>
</template>

<script setup>
import Button from 'primevue/button'
import Paginator from 'primevue/paginator'
import VisitLogList from '@/components/visitLog/VisitLogList.vue'
import { ref, onMounted, computed, nextTick } from 'vue'
import { getMyVisitLog } from '@/api/visitLog'
import VisitPrefModal from '@/components/visitLog/VisitPrefModal.vue'
import { useVisitPrefController } from '@/composables/visit/useVisitPrefController'
import dayjs from 'dayjs'
import relativeTime from 'dayjs/plugin/relativeTime'
import duration from 'dayjs/plugin/duration'
import 'dayjs/locale/ko'
import VisitMap from '@/components/visitLog/VisitMap.vue'

dayjs.extend(relativeTime)
dayjs.extend(duration)
dayjs.locale('ko')

const page = ref(1)
const logs = ref([])
const totalCount = ref(0)
const visitPrefVisible = ref(false)
const visitPrefId = ref(null)
const visitPrefTitle = ref('')
const visitPrefScore = ref(null)

const fetchLogs = async () => {
  try {
    const res = await getMyVisitLog(page.value)
    logs.value = res.data.visitLogs.reverse()
    totalCount.value = res.data.totalCount
    await nextTick()
    window.scrollTo({ top: 0, behavior: 'auto' })
  } catch (err) {
    console.error('방문기록 불러오기 실패:', err)
  }
}

const visitSummary = computed(() => {
  const count = logs.value.length
  return count > 0 ? `총 ${count}개의 관광지를 방문했어요.` : ''
})


onMounted(fetchLogs)

const onPageChange = (event) => {
  page.value = event.page + 1
  fetchLogs()
}

const goToPrevPage = () => {
  if (page.value > 1) {
    page.value--
    fetchLogs()
  }
}

const goToNextPage = () => {
  if (page.value < totalCount.value) {
    page.value++
    fetchLogs()
  }
}

const handlePreferenceClick = (log) => {
  visitPrefId.value = log.visitLogId
  visitPrefTitle.value = log.title
  visitPrefScore.value = log.preference ?? null
  visitPrefVisible.value = true
}
const { handleSubmit: handlePrefSubmit } = useVisitPrefController(
  visitPrefVisible,
  visitPrefId,
  visitPrefTitle,
  fetchLogs
)
</script>
