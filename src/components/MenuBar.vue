<template>
  <nav
    class="fixed border-gray-200 z-50 h-16 w-full bottom-0 border-t md:h-full md:top-0 md:right-0 md:bottom-auto md:border-none md:w-1/6 bg-white"
  >
    <ul
      class="flex w-full h-full items-center justify-around md:flex-col md:justify-center md:space-y-6"
    >
      <li v-for="item in navItems" :key="item.path">
        <RouterLink
          :to="item.path"
          class="flex flex-col items-center justify-center text-gray-400 cursor-pointer"
          active-class="text-primary"
          exact-active-class="text-primary"
        >
          <!-- 모바일: 하단 아이콘 -->
          <div class="md:hidden">
            <i :class="`pi ${item.icon}`" style="font-size: 24px"></i>
          </div>

          <!-- 데스크탑: 오른쪽 세로 버튼형 아이콘 -->
          <div
            class="hidden md:flex items-center justify-center p-4 border-l-gray-200 border border-gray-200 rounded-xl md:hover:bg-gray-100 transition-colors"
          >
            <i :class="`pi ${item.icon}`" style="font-size: 28px"></i>
          </div>
        </RouterLink>
      </li>
    </ul>
  </nav>
</template>

<script setup>
import { RouterLink } from 'vue-router'

const navItems = [
  { path: '/', icon: 'pi-map' },
  { path: '/visit-log', icon: 'pi-calendar-clock' },
  { path: '/mypage', icon: 'pi-user' },
  { path: '/setting', icon: 'pi-ellipsis-h' }
]
</script>
