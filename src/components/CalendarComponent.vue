<template>
  <div class="min-h-screen bg-gray-100 flex flex-col">
    <div class="flex-grow flex flex-col max-w-md w-full mx-auto bg-white shadow-lg overflow-hidden transition-all duration-300 ease-in-out">
      <!-- Calendar Header -->
      <div
          class="bg-gray-800 p-6 text-white transition-all duration-300 ease-in-out"
          :class="{ 'pb-8': isCalendarExpanded, 'pb-4': !isCalendarExpanded }"
      >
        <div class="flex justify-between items-center mb-6">
          <button
              @click="changeMonth(-1)"
              class="text-white hover:text-white-200 transition-colors"
          >
            <ChevronLeftIcon class="h-6 w-6" />
          </button>
          <h2 class="text-xl font-semibold">{{ currentMonth }} {{ currentYear }}</h2>
          <button
              @click="changeMonth(1)"
              class="text-white hover:text-white-200 transition-colors"
          >
            <ChevronRightIcon class="h-6 w-6" />
          </button>
        </div>

        <!-- Calendar Grid -->
        <transition name="fade" mode="out-in">
          <div v-if="isCalendarExpanded" :key="currentMonth + currentYear">
            <div class="grid grid-cols-7 gap-2 mb-4 text-gray-400 text-sm">
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
                  'text-gray-300': !date.isCurrentMonth,
                  'text-white': date.isCurrentMonth,
                  'bg-gray-700': date.isSelected,
                  'hover:bg-gray-600': date.isCurrentMonth && !date.isSelected
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
            <div class="grid grid-cols-4 gap-4 text-sm text-gray-200 mt-4">
              <div class="flex items-center space-x-2">
                <span class="w-2 h-2 bg-red-400 transform rotate-45"></span>
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
              class="w-12 h-12 bg-gray-600 rounded-full flex items-center justify-center transition-transform duration-300 hover:bg-gray-700"
              :class="{ 'rotate-180': !isCalendarExpanded }"
          >
            <ChevronUpIcon class="h-8 w-8 text-white" />
          </div>
        </div>
        <div class="h-px bg-gray-100"></div>
      </div>

      <!-- Selected Date Information -->
      <div class="flex-grow overflow-y-auto p-6 transition-all duration-300 ease-in-out">
        <h3 class="text-lg font-semibold text-gray-900 mb-4">
          {{ selectedDate ? formatDate(selectedDate.datum) : 'Wählen Sie ein Datum' }}
        </h3>
        <div v-if="selectedDate" class="space-y-4">
          <div class="p-4 rounded-xl bg-gray-50">
            <h4 class="font-medium text-gray-900">Fasteninformation</h4>
            <p class="text-sm text-gray-600">{{ selectedDate.fastenInformation }}</p>
          </div>
          <div v-if="selectedDate.evangelium" @click="fetchBibleText('evangelium')" class="p-4 rounded-xl bg-gray-50 cursor-pointer hover:bg-gray-100 transition-colors">
            <h4 class="font-medium text-gray-900">Evangelium</h4>
            <p class="text-sm text-gray-600">{{ selectedDate.evangelium }}</p>
          </div>
          <div v-if="selectedDate.epistel" @click="fetchBibleText('epistel')" class="p-4 rounded-xl bg-gray-50 cursor-pointer hover:bg-gray-100 transition-colors">
            <h4 class="font-medium text-gray-900">Epistel</h4>
            <p class="text-sm text-gray-600">{{ selectedDate.epistel }}</p>
          </div>
          <div v-if="selectedDate.orthrosEvangelium" @click="fetchBibleText('orthrosEvangelium')" class="p-4 rounded-xl bg-gray-50 cursor-pointer hover:bg-gray-100 transition-colors">
            <h4 class="font-medium text-gray-900">Orthros Evangelium</h4>
            <p class="text-sm text-gray-600">{{ selectedDate.orthrosEvangelium }}</p>
          </div>
          <div v-if="selectedDate.altesTestament" @click="fetchBibleText('altesTestament')" class="p-4 rounded-xl bg-gray-50 cursor-pointer hover:bg-gray-100 transition-colors">
            <h4 class="font-medium text-gray-900">Altes Testament</h4>
            <p class="text-sm text-gray-600">{{ selectedDate.altesTestament }}</p>
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

<script setup>
import { ref, computed, watch } from 'vue'
import { ChevronUpIcon, ChevronLeftIcon, ChevronRightIcon } from 'lucide-vue-next'

// State
const isCalendarExpanded = ref(true)
const currentDate = ref(new Date())
const monthData = ref([])
const selectedDate = ref(null)
const bibleText = ref(null)

// Toggle calendar expansion
const toggleCalendar = () => {
  isCalendarExpanded.value = !isCalendarExpanded.value
}

// Month navigation
const changeMonth = async (delta) => {
  currentDate.value = new Date(currentDate.value.getFullYear(), currentDate.value.getMonth() + delta, 1)
  await fetchMonthData()
}

// Fetch month data
const fetchMonthData = async () => {
  const year = currentDate.value.getFullYear();
  const month = currentDate.value.getMonth() + 1; // JavaScript months are 0-indexed
  try {
    const response = await fetch(`https://wmano.uber.space/api/calendar/public/getMonth?year=${year}&month=${month}`);
    const data = await response.json();
    monthData.value = data;

    // Set the selected date to today if it's in the current month
    const today = new Date();
    const todayData = data.find(d => {
      const datum = new Date(d.datum);
      return (
          datum.getDate() === today.getDate() &&
          datum.getMonth() === today.getMonth() &&
          datum.getFullYear() === today.getFullYear()
      );
    });

    if (todayData) {
      selectedDate.value = todayData;
    }
  } catch (error) {
    console.error('Error fetching month data:', error);
  }
};

// Select a date
const selectDate = (date) => {
  const selectedDateData = monthData.value.find(d => {
    const dDate = new Date(d.datum)
    return dDate.getDate() === date.day && dDate.getMonth() === date.month && dDate.getFullYear() === date.year
  })
  selectedDate.value = selectedDateData
  bibleText.value = null
}

// Format date
const formatDate = (dateString) => {
  const date = new Date(dateString)
  return date.toLocaleDateString('de-DE', { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' })
}

// Determine marker color based on fastenInformation
const getMarkerColor = (fastenInformation) => {
  if (fastenInformation.includes('Strikter Fastentag. Enthaltung von Fleisch, Fisch, Öl, Wein, Milchprodukte und Eiern')) return 'black'
  if (fastenInformation.includes('Wein und Öl ist erlaubt. Enthaltung von Fleisch, Fisch, Milchprodukte und Eiern.')) return 'red'
  if (fastenInformation.includes('Fisch, Öl und Wein ist erlaubt. Enthaltung von Fleisch, Milchprodukte und Eiern.')) return 'blue'
  if (fastenInformation.includes('Milchprodukte, Fisch, Öl und Wein ist erlaubt. Enthaltung von Fleisch')) return 'yellow'
  return null
}

// Fetch Bible text
const fetchBibleText = async (key) => {
  if (!selectedDate.value || !selectedDate.value[key]) return

  try {
    const response = await fetch('https://wmano.uber.space/api/bibleText/public/getByTextstelle', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({ 'text': selectedDate.value[key] }),
    })
    const data = await response.json()
    bibleText.value = data.text
  } catch (error) {
    console.error('Error fetching Bible text:', error)
    bibleText.value = 'Fehler beim Laden des Bibeltextes.'
  }
}

// Calendar days computation
const calendarDays = computed(() => {
  const days = []
  const year = currentDate.value.getFullYear()
  const month = currentDate.value.getMonth()

  const firstDay = new Date(year, month, 1)
  const lastDay = new Date(year, month + 1, 0)

  // Add previous month's days
  const firstDayOfWeek = firstDay.getDay() || 7
  for (let i = firstDayOfWeek - 1; i > 0; i--) {
    const day = new Date(year, month, 1 - i)
    days.push({
      day: day.getDate(),
      month: day.getMonth(),
      year: day.getFullYear(),
      isCurrentMonth: false,
      isSelected: false,
      marker: null
    })
  }

  // Add current month's days
  for (let day = 1; day <= lastDay.getDate(); day++) {
    const date = new Date(year, month, day)
    const dateData = monthData.value.find(d => new Date(d.datum).getDate() === day)
    days.push({
      day,
      month,
      year,
      isCurrentMonth: true,
      isSelected: selectedDate.value && new Date(selectedDate.value.datum).getDate() === day,
      marker: dateData ? getMarkerColor(dateData.fastenInformation) : null
    })
  }

  // Add next month's days to fill the calendar
  const remainingDays = 42 - days.length // 6 rows * 7 days = 42
  for (let i = 1; i <= remainingDays; i++) {
    const day = new Date(year, month + 1, i)
    days.push({
      day: day.getDate(),
      month: day.getMonth(),
      year: day.getFullYear(),
      isCurrentMonth: false,
      isSelected: false,
      marker: null
    })
  }

  return days
})

// Computed properties
const currentMonth = computed(() => {
  return currentDate.value.toLocaleString('de-DE', { month: 'long' })
})

const currentYear = computed(() => {
  return currentDate.value.getFullYear()
})

// Fetch initial data
fetchMonthData()

// Watch for changes in currentDate and refetch data
watch(currentDate, fetchMonthData)
</script>

<style scoped>
.collapse-enter-active,
.collapse-leave-active {
  transition: all 0.3s ease-in-out;
  max-height: 500px;
}

.collapse-enter-from,
.collapse-leave-to {
  max-height: 0;
  opacity: 0;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>