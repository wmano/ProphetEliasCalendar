<template>
  <div class="min-h-screen bg-purple-50 flex flex-col">
    <div class="flex-grow flex flex-col max-w-md w-full mx-auto bg-white shadow-lg overflow-hidden transition-all duration-300 ease-in-out">
      <!-- Calendar Header -->
      <div
          class="bg-purple-800 p-6 text-white transition-all duration-300 ease-in-out"
          :class="{ 'pb-8': isCalendarExpanded, 'pb-4': !isCalendarExpanded }"
      >
        <div class="flex justify-between items-center mb-6">
          <button
              @click="changeMonth(-1)"
              class="text-white hover:text-purple-200 transition-colors"
          >
            <ChevronLeftIcon class="h-6 w-6" />
          </button>
          <h2 class="text-xl font-semibold">{{ currentMonth }} {{ currentYear }}</h2>
          <button
              @click="changeMonth(1)"
              class="text-white hover:text-purple-200 transition-colors"
          >
            <ChevronRightIcon class="h-6 w-6" />
          </button>
        </div>

        <!-- Calendar Grid -->
        <transition name="fade" mode="out-in">
          <div v-if="isCalendarExpanded" :key="currentMonth + currentYear">
            <div class="grid grid-cols-7 gap-2 mb-4 text-purple-200 text-sm">
              <span v-for="day in ['Mo', 'Tu', 'We', 'Th', 'Fr', 'Sa', 'Su']" :key="day" class="text-center">
                {{ day }}
              </span>
            </div>
            <div class="grid grid-cols-7 gap-2 mb-6">
              <button
                  v-for="date in calendarDays"
                  :key="date.day + date.month + date.year"
                  @click="selectDate(date)"
                  class="aspect-square rounded-full flex items-center justify-center text-sm relative"
                  :class="{
                  'text-purple-300': !date.isCurrentMonth,
                  'text-white': date.isCurrentMonth,
                  'bg-purple-700': date.isSelected,
                  'hover:bg-purple-600': date.isCurrentMonth && !date.isSelected
                }"
              >
                {{ date.day }}
                <span
                    v-if="date.marker"
                    class="absolute -bottom-1 w-2 h-2 rounded-full"
                    :class="{
                    'bg-black': date.marker === 'black',
                    'bg-red-400': date.marker === 'red',
                    'bg-blue-400': date.marker === 'blue',
                    'bg-yellow-400': date.marker === 'yellow'
                  }"
                ></span>
              </button>
            </div>

            <!-- Legend -->
            <div class="grid grid-cols-4 gap-4 text-sm text-purple-200 mt-4">
              <div class="flex items-center space-x-2">
                <span class="w-2 h-2 rounded-full bg-black"></span>
                <span>Streng</span>
              </div>
              <div class="flex items-center space-x-2">
                <span class="w-2 h-2 rounded-full bg-red-400"></span>
                <span>+ Öl</span>
              </div>
              <div class="flex items-center space-x-2">
                <span class="w-2 h-2 rounded-full bg-blue-400"></span>
                <span>+ Fisch</span>
              </div>
              <div class="flex items-center space-x-2">
                <span class="w-2 h-2 rounded-full bg-yellow-400"></span>
                <span>+ Milch</span>
              </div>
            </div>
          </div>
        </transition>
      </div>

      <!-- Divider with Toggle Button -->
      <div class="relative">
        <div
            class="absolute left-1/2 -translate-x-1/2 -translate-y-1/2 z-10 bg-white rounded-full shadow-lg p-1 cursor-pointer"
            @click="toggleCalendar"
        >
          <div
              class="w-12 h-12 bg-green-500 rounded-full flex items-center justify-center transition-transform duration-300 hover:bg-green-600"
              :class="{ 'rotate-180': !isCalendarExpanded }"
          >
            <ChevronUpIcon class="h-8 w-8 text-white" />
          </div>
        </div>
        <div class="h-px bg-purple-100"></div>
      </div>

      <!-- Selected Date Information -->
      <div class="flex-grow overflow-y-auto p-6 transition-all duration-300 ease-in-out">
        <h3 class="text-lg font-semibold text-purple-900 mb-4">
          {{ selectedDate ? formatDate(selectedDate.datum) : 'Wählen Sie ein Datum' }}
        </h3>
        <div v-if="selectedDate" class="space-y-4">
          <div class="p-4 rounded-xl bg-purple-50">
            <h4 class="font-medium text-purple-900">Fasteninformation</h4>
            <p class="text-sm text-purple-600">{{ selectedDate.fastenInformation }}</p>
          </div>
          <div v-if="selectedDate.evangelium" @click="fetchBibleText('evangelium')" class="p-4 rounded-xl bg-purple-50 cursor-pointer hover:bg-purple-100 transition-colors">
            <h4 class="font-medium text-purple-900">Evangelium</h4>
            <p class="text-sm text-purple-600">{{ selectedDate.evangelium }}</p>
          </div>
          <div v-if="selectedDate.epistel" @click="fetchBibleText('epistel')" class="p-4 rounded-xl bg-purple-50 cursor-pointer hover:bg-purple-100 transition-colors">
            <h4 class="font-medium text-purple-900">Epistel</h4>
            <p class="text-sm text-purple-600">{{ selectedDate.epistel }}</p>
          </div>
          <div v-if="selectedDate.orthrosEvangelium" @click="fetchBibleText('orthrosEvangelium')" class="p-4 rounded-xl bg-purple-50 cursor-pointer hover:bg-purple-100 transition-colors">
            <h4 class="font-medium text-purple-900">Orthros Evangelium</h4>
            <p class="text-sm text-purple-600">{{ selectedDate.orthrosEvangelium }}</p>
          </div>
          <div v-if="selectedDate.altesTestament" @click="fetchBibleText('altesTestament')" class="p-4 rounded-xl bg-purple-50 cursor-pointer hover:bg-purple-100 transition-colors">
            <h4 class="font-medium text-purple-900">Altes Testament</h4>
            <p class="text-sm text-purple-600">{{ selectedDate.altesTestament }}</p>
          </div>
        </div>
        <!-- Bible Text Display -->
        <div v-if="bibleText" class="mt-6 p-4 rounded-xl bg-blue-50">
          <h4 class="font-medium text-blue-900 mb-2">Bibeltext</h4>
          <p class="text-sm text-blue-600">{{ bibleText }}</p>
        </div>
      </div>
    </div>
  </div>
</template>
<script setup lang="ts">
import {ChevronLeftIcon, ChevronRightIcon, ChevronUpIcon} from "lucide-vue-next";
</script>