<script setup>
import { ref, computed } from 'vue'

const level = ref(0)
const star4 = ref(0)
const star3 = ref(0)
const star2 = ref(0)

const useCustomStar4 = ref(false)
const customStar4Need = ref(0)

const star4Exp = 20000
const star3Exp = 5000
const star2Exp = 1000

const levels = [
  {
    name: 'Level 1 to 20',
    exp: 112510,
  },
  {
    name: 'Level 20 to 30',
    exp: 177910,
  },
  {
    name: 'Level 30 to 40',
    exp: 206290,
  },
  {
    name: 'Level 40 to 50',
    exp: 389390,
  },
  {
    name: 'Level 50 to 60',
    exp: 822140,
  },
  {
    name: 'Level 60 to 70',
    exp: 1326050,
  },
  {
    name: 'Level 70 to 80',
    exp: 2763000,
  },
]

const expNeeded = computed(() => {
  return levels[level.value].exp
})

const star2NeedBase = computed(() => {
  if (star2.value == 0) return 0

  const ratio32 = star3.value / star2.value
  const ratio42 = star4.value / star2.value

  if (useCustomStar4.value) {
    const remainingExp = expNeeded.value - star4Exp * star4Need.value
    if (remainingExp < 0) return 0

    return Math.floor(remainingExp  / (star3Exp * ratio32 + star2Exp))
  }

  return Math.floor(expNeeded.value / (star4Exp * ratio42 + star3Exp * ratio32 + star2Exp))
})

const star2Need = computed(() => {
  const remainingExp = expNeeded.value - star4Exp * star4Need.value - star3Exp * star3NeedBase.value - star2Exp * star2NeedBase.value
  const remainingStar2Need = Math.ceil((remainingExp % star3Exp) / star2Exp)
  if (remainingStar2Need < 0) return star2NeedBase.value

  return star2NeedBase.value + remainingStar2Need
})

const star3NeedBase = computed(() => {
  if (star3.value == 0) return 0

  const ratio23 = star2.value / star3.value
  const ratio43 = star4.value / star3.value

  if (useCustomStar4.value) {
    const remainingExp = expNeeded.value - star4Exp * star4Need.value
    if (remainingExp < 0) return 0
    
    return Math.floor(remainingExp / (star3Exp + star2Exp * ratio23))
  }

  return Math.floor(expNeeded.value / (star4Exp * ratio43 + star3Exp + star2Exp * ratio23))
})

const star3Need = computed(() => {
  const remainingExp = expNeeded.value - star4Exp * star4Need.value - star3Exp * star3NeedBase.value - star2Exp * star2NeedBase.value
  const remainingStar3Need = Math.floor(remainingExp / star3Exp)
  if (remainingStar3Need < 0) return star3NeedBase.value
  
  return star3NeedBase.value + remainingStar3Need
})

const star4Need = computed({
  get: () => {
    const ratio24 = star2.value / star4.value
    const ratio34 = star3.value / star4.value

    if (useCustomStar4.value) return customStar4Need.value
    if (star4.value == 0) return 0
    return Math.floor(expNeeded.value / (star4Exp + star3Exp * ratio34 + star2Exp * ratio24))
  },
  set: newValue => customStar4Need.value = newValue,
})
</script>

<template>
  <div class="mx-auto max-w-sm min-h-screen shadow-lg
    bg-purple-100 px-4 py-8">
    <h1 class="text-2xl font-semibold text-gray-900">
      Star Rail Level Up Calculator
    </h1>

    <select v-model="level" class="mt-4 block w-full text-gray-900">
      <option v-for="(level, index) in levels" :key="index" :value="index">
        {{ `${level.name} (${level.exp} EXP)` }}
      </option>
    </select>

    <h2 class="mt-8 text-xl font-semibold text-gray-900">Materials Owned</h2>

    <div class="flex gap-4">
      <div class="flex-1">
        <img src="./assets/img/star-4-travelers-guide.png"
          alt="Traveler's Guide" class="p-2" />
        <input type="number" min="0" v-model="star4"
          class="block w-full text-gray-900 text-center" />
      </div>
      <div class="flex-1">
        <img src="./assets/img/star-3-adventure-log.png"
          alt="Adventure Log" class="p-2" />
        <input type="number" min="0" v-model="star3"
          class="block w-full text-gray-900 text-center" />
      </div>
      <div class="flex-1">
        <img src="./assets/img/star-2-travel-encounters.png"
          alt="Travel Encounters" class="p-2" />
        <input type="number" min="0" v-model="star2"
          class="block w-full text-gray-900 text-center" />
      </div>
    </div>

    <h2 class="mt-8 text-xl font-semibold text-gray-900">Materials Needed</h2>

    <div class="flex gap-4">
      <div class="flex-1">
        <img src="./assets/img/star-4-travelers-guide.png"
          alt="Traveler's Guide" class="p-2" />
        <input type="number" min="0"
          :disabled="!useCustomStar4" v-model="star4Need"
          :class="['block w-full border-2 font-semibold text-center',
          star4Need > star4 ? 'text-red-600' : 'text-purple-600',
          useCustomStar4 ? 'bg-white' : 'border-purple-600 bg-purple-100']" />
      </div>
      <div class="flex-1">
        <img src="./assets/img/star-3-adventure-log.png"
          alt="Adventure Log" class="p-2" />
        <input type="number" min="0" disabled v-model="star3Need"
        :class="['block w-full border-2 border-purple-600',
        'bg-purple-100 font-semibold text-center',
        star3Need > star3 ? 'text-red-600' : 'text-purple-600']" />
      </div>
      <div class="flex-1">
        <img src="./assets/img/star-2-travel-encounters.png"
          alt="Travel Encounters" class="p-2" />
        <input type="number" min="0" disabled v-model="star2Need"
          :class="['block w-full border-2 border-purple-600',
          'bg-purple-100 font-semibold text-center',
          star2Need > star2 ? 'text-red-600' : 'text-purple-600']" />
      </div>
    </div>

    <label class="mt-4 block">
      <input type="checkbox" v-model="useCustomStar4" />
      <span class="ml-2 text-gray-900">Adjust 4-star</span>
    </label>
  </div>
</template>