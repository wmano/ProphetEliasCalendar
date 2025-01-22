<template>
  <div class="min-h-screen bg-gray-100 flex flex-col">
    <div class="flex-grow flex flex-col max-w-md w-full mx-auto bg-white shadow-lg overflow-hidden transition-all duration-300 ease-in-out">
      <!-- Calendar Header -->
      <div
          :class="[
          'bg-gray-800 p-6 text-white transition-all duration-300 ease-in-out',
          isCalendarExpanded ? 'pb-8' : 'pb-4'
        ]"
      >
        <div class="flex justify-between items-center mb-6">
          <button
              @click="changeMonth(-1)"
              class="text-white hover:text-white transition-colors"
          >
            <ChevronLeftIcon class="h-6 w-6" />
          </button>
          <h2 class="text-xl font-semibold">
            {{ currentDate.toLocaleString('de-DE', { month: 'long' }) }}
            {{ currentDate.getFullYear() }}
          </h2>
          <button
              @click="changeMonth(1)"
              class="text-white hover:text-white transition-colors"
          >
            <ChevronRightIcon class="h-6 w-6" />
          </button>
        </div>

        <!-- Calendar Grid -->
        <div v-if="isCalendarExpanded" :key="`${currentDate.getMonth()}${currentDate.getFullYear()}`">
          <div class="grid grid-cols-7 gap-2 mb-4 text-gray-400 text-sm">
            <span v-for="day in ['Mo', 'Tu', 'We', 'Th', 'Fr', 'Sa', 'Su']" :key="day" class="text-center">
              {{ day }}
            </span>
          </div>
          <div class="grid grid-cols-7 gap-2 mb-6">
            <button
                v-for="(date, index) in calendarDays"
                :key="index"
                @click="selectDate(date)"
                :class="[
                'aspect-square rounded-full flex items-center justify-center text-sm relative',
                !date.isCurrentMonth ? 'text-gray-500' : 'text-white',
                date.isSelected ? 'bg-gray-600' : '',
                date.isCurrentMonth && !date.isSelected ? 'hover:bg-gray-700' : ''
              ]"
            >
              {{ date.day }}
              <span
                  v-if="date.marker"
                  :class="['absolute -bottom-1 w-2 h-2 rounded-full', date.marker]"
              ></span>
            </button>
          </div>

          <!-- Legend -->
          <div class="grid grid-cols-4 gap-4 text-sm text-gray-400 mt-4">
            <div v-for="(legend, index) in legends" :key="index" class="flex items-center space-x-2">
              <span :class="['w-2 h-2 rounded-full', legend.color]"></span>
              <span>{{ legend.label }}</span>
            </div>
          </div>
        </div>
      </div>

      <!-- Divider with Toggle Button -->
      <div class="relative">
        <div
            class="absolute left-1/2 -translate-x-1/2 -translate-y-1/2 z-10 bg-white rounded-full shadow-lg p-1 cursor-pointer"
            @click="toggleCalendar"
        >
          <div
              :class="[
              'w-12 h-12 bg-gray-600 rounded-full flex items-center justify-center transition-transform duration-300 hover:bg-gray-700',
              !isCalendarExpanded ? 'rotate-180' : ''
            ]"
          >
            <ChevronUpIcon class="h-8 w-8 text-white" />
          </div>
        </div>
        <div class="h-px bg-gray-200"></div>
      </div>

      <!-- Selected Date Information -->
      <div class="flex-grow overflow-y-auto p-6 transition-all duration-300 ease-in-out">
        <h3 class="text-lg font-semibold text-gray-800 mb-4">
          {{ selectedDate ? formatDate(selectedDate.datum) : 'Wählen Sie ein Datum' }}
        </h3>
        <div v-if="selectedDate" class="space-y-4">
          <div class="p-4 rounded-xl bg-gray-100">
            <h4 class="font-medium text-gray-800">Fasteninformation</h4>
            <p class="text-sm text-gray-600">{{ selectedDate.fastenInformation }}</p>
          </div>
          <div
              v-for="(item, key) in selectedDateItems"
              :key="key"
              @click="fetchBibleText(key)"
              class="p-4 rounded-xl bg-gray-100 cursor-pointer hover:bg-gray-200 transition-colors"
          >
            <h4 class="font-medium text-gray-800">{{ item.title }}</h4>
            <p class="text-sm text-gray-600">{{ item.text }}</p>
          </div>
        </div>
        <!-- Bible Text Display -->
        <div v-if="bibleText" class="mt-6 p-4 rounded-xl bg-gray-200">
          <h4 class="font-medium text-gray-800 mb-2">Bibeltext</h4>
          <p class="text-sm text-gray-600">{{ bibleText }}</p>
        </div>
      </div>
    </div>
  </div>
</template>
<script setup lang="ts">
import {ChevronLeftIcon, ChevronRightIcon, ChevronUpIcon} from "lucide-vue-next";
</script>