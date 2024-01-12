<script setup>
import { ref, computed } from 'vue'

import chLevels from './data/characterLevels'
import lcLevels from './data/lightConeLevels'
import gChLevels from './data/genshinCharacterLevels'
import chMaterials from './data/characterMaterials'
import lcMaterials from './data/lightConeMaterials'
import gChMaterials from './data/genshinCharacterMaterials'

const modes = ['Character', 'Light Cone', 'Genshin']

/**
 * Use ratio for the amount of materials needed or default.
 */
const useRatio = ref(false)

/**
 * Use custom 4-star amount or default.
 */
 const useCustomStar4 = ref(false)

/**
 * Use custom 3-star amount or default.
 */
const useCustomStar3 = ref(false)

/**
 * Use custom 2-star amount or default.
 */
const useCustomStar2 = ref(false)

/**
 * Index of mode: Character or Light Cone.
 */
 const modeIndex = ref(0)

/**
 * Selected Light Cone star.
 */
const lcStar = ref(5)

/**
 * Index of selected level.
 */
 const levelIndex = ref(0)

const star4Ratio = ref(1)
const star3Ratio = ref(1)
const star2Ratio = ref(1)

/**
 * Amount of custom 4-star materials needed.
 */
 const customStar4Need = ref(null)

/**
 * Amount of custom 3-star materials needed.
 */
const customStar3Need = ref(null)

/**
 * Amount of custom 2-star materials needed.
 */
const customStar2Need = ref(null)

/**
 * Exp of a 4-star material (Character or Light Cone).
 */
const star4Exp = computed(() => {
  switch (modeIndex.value) {
    case 0:
      return chMaterials[0].exp
    case 1:
      return lcMaterials[0].exp
    case 2:
      return gChMaterials[0].exp
    default:
      return chMaterials[0].exp
  }
})

/**
 * Exp of a 3-star material (Character or Light Cone).
 */
const star3Exp = computed(() => {
  switch (modeIndex.value) {
    case 0:
      return chMaterials[1].exp
    case 1:
      return lcMaterials[1].exp
    case 2:
      return gChMaterials[1].exp
    default:
      return chMaterials[1].exp
  }
})

/**
 * Exp of a 2-star material (Character or Light Cone).
 */
const star2Exp = computed(() => {
  switch (modeIndex.value) {
    case 0:
      return chMaterials[2].exp
    case 1:
      return lcMaterials[2].exp
    case 2:
      return gChMaterials[2].exp
    default:
      return chMaterials[2].exp
  }
})

/**
 * Exp requirement of all levels.
 */
const levels = computed(() => {
  if (modeIndex.value == 0) return chLevels

  if (modeIndex.value == 1) {
    return lcLevels.map(level => {
      return {
        name: level.name,
        exp: level.expList[lcStar.value - 3]
      }
    })
  }

  if (modeIndex.value == 2) return gChLevels
})

/**
 * Exp requirement of selected level.
 */
const expNeeded = computed(() => {
  return levels.value[levelIndex.value].exp
})

/**
 * Final amount of 4-star materials needed.
 */
const star4Need = computed({
  get: () => {
    if (useRatio.value) {
      if (star4Ratio.value == 0) return 0

      const ratio24 = star2Ratio.value / star4Ratio.value
      const ratio34 = star3Ratio.value / star4Ratio.value

      return Math.floor(expNeeded.value / (star4Exp.value + star3Exp.value * ratio34 + star2Exp.value * ratio24))
    }

    let expRemainder = expNeeded.value

    if (useCustomStar4.value) return customStar4Need.value

    if (useCustomStar3.value) {
      expRemainder -= customStar3Need.value * star3Exp.value
    }

    if (useCustomStar2.value) {
      expRemainder -= customStar2Need.value * star2Exp.value
    }

    expRemainder = expRemainder < 0 ? 0 : expRemainder
    
    return Math.floor(expRemainder / star4Exp.value)
  },
  set: newValue => {
    useCustomStar4.value = true
    useCustomStar3.value = false
    useCustomStar2.value = false

    customStar4Need.value = newValue
  },
})

/**
 * Temporary amount of 3-star materials needed.
 */
const star3NeedRatioBase = computed(() => {
  if (star3Ratio.value == 0) return 0

  const ratio23 = star2Ratio.value / star3Ratio.value
  const ratio43 = star4Ratio.value / star3Ratio.value

  return Math.floor(expNeeded.value / (star4Exp.value * ratio43 + star3Exp.value + star2Exp.value * ratio23))
})

/**
 * Final amount of 3-star materials needed.
 */
const star3Need = computed({
  get: () => {
    let expRemainder = expNeeded.value

    if (useRatio.value) {
      expRemainder -= star4Exp.value * star4Need.value
      expRemainder -= star3Exp.value * star3NeedRatioBase.value
      expRemainder -= star2Exp.value * star2NeedRatioBase.value
      
      const remainingStar3Need = Math.floor(expRemainder / star3Exp.value)
      if (remainingStar3Need < 0) return star3NeedRatioBase.value

      return star3NeedRatioBase.value + remainingStar3Need
    }

    if (useCustomStar4.value) {
      expRemainder -= customStar4Need.value * star4Exp.value
      expRemainder = expRemainder < 0 ? 0 : expRemainder
      
      return Math.floor(expRemainder / star3Exp.value)
    }

    if (useCustomStar3.value) return customStar3Need.value

    if (useCustomStar2.value) {
      expRemainder -= customStar2Need.value * star2Exp.value
      expRemainder = expRemainder < 0 ? 0 : expRemainder

      expRemainder %= star4Exp.value
      return Math.ceil(expRemainder / star3Exp.value)
    }

    expRemainder %= star4Exp.value
    return Math.floor(expRemainder / star3Exp.value)
  },
  set: newValue => {
    useCustomStar4.value = false
    useCustomStar3.value = true
    useCustomStar2.value = false

    customStar3Need.value = newValue
  },
})

/**
 * Temporary amount of 2-star materials needed.
 */
const star2NeedRatioBase = computed(() => {
  if (star2Ratio.value == 0) return 0

  const ratio32 = star3Ratio.value / star2Ratio.value
  const ratio42 = star4Ratio.value / star2Ratio.value

  return Math.floor(expNeeded.value / (star4Exp.value * ratio42 + star3Exp.value * ratio32 + star2Exp.value))
})

/**
 * Final amount of 2-star materials needed.
 */
const star2Need = computed({
  get: () => {
    let expRemainder = expNeeded.value

    if (useRatio.value) {
      expRemainder -= star4Exp.value * star4Need.value
      expRemainder -= star3Exp.value * star3NeedRatioBase.value
      expRemainder -= star2Exp.value * star2NeedRatioBase.value
      
      const remainingStar2Need = Math.ceil((expRemainder % star3Exp.value) / star2Exp.value)
      if (remainingStar2Need < 0) return star2NeedRatioBase.value

      return star2NeedRatioBase.value + remainingStar2Need
    }

    if (useCustomStar4.value) {
      expRemainder -= customStar4Need.value * star4Exp.value
      expRemainder = expRemainder < 0 ? 0 : expRemainder

      expRemainder %= star3Exp.value
      return Math.ceil(expRemainder / star2Exp.value)
    }

    if (useCustomStar3.value) {
      expRemainder -= customStar3Need.value * star3Exp.value
      expRemainder = expRemainder < 0 ? 0 : expRemainder

      expRemainder %= star4Exp.value
      return Math.ceil(expRemainder / star2Exp.value)
    }

    if (useCustomStar2.value) return customStar2Need.value

    expRemainder %= star4Exp.value
    expRemainder %= star3Exp.value
    return Math.ceil(expRemainder / star2Exp.value)
  },
  set: newValue => {
    useCustomStar4.value = false
    useCustomStar3.value = false
    useCustomStar2.value = true

    customStar2Need.value = newValue
  },
})

const setModeIndex = (index) => modeIndex.value = index

/**
 * Reset the custom materials flags.
 */
const resetCustom = () => {
  useCustomStar4.value = false
  useCustomStar3.value = false
  useCustomStar2.value = false
}

/**
 * Get the image of a Character or Light Cone material.
 * @param {Number} index Index of material.
 */
const getImage = (index) => {
  switch (modeIndex.value) {
    case 0:
      return chMaterials[index].image
    case 1:
      return lcMaterials[index].image
    case 2:
      return gChMaterials[index].image
    default:
      return chMaterials[index].image
  }
}

/**
 * Get the name of a Character or Light Cone material.
 * @param {Number} index 
 */
const getImageName = (index) => {
  switch (modeIndex.value) {
    case 0:
      return chMaterials[index].name
    case 1:
      return lcMaterials[index].name
    case 2:
      return gChMaterials[index].name
    default:
      return chMaterials[index].name
  }
}
</script>

<template>
  <div class="mx-auto max-w-sm min-h-screen shadow-lg
    bg-purple-100 px-4 py-8">
    <h1 class="text-2xl font-semibold text-gray-900">
      Star Rail Level Up Calculator
    </h1>

    <div class="mt-4 flex justify-evenly">
      <a v-for="(mode, index) in modes" :key="index"
        href="#" @click.prevent="setModeIndex(index)"
        :class="['font-semibold', modeIndex == index
        ? 'text-purple-600 underline underline-offset-8 decoration-2'
        : 'text-purple-500 hover:text-purple-600']">
        {{ mode }}
      </a>
    </div>

    <div class="mt-4">
      <select v-if="modeIndex == 1" v-model="lcStar"
        class="block w-full text-gray-800">
        <option value="5">5-Star Light Cone</option>
        <option value="4">4-Star Light Cone</option>
        <option value="3">3-Star Light Cone</option>
      </select>

      <select v-model="levelIndex" @change="resetCustom()"
        class="mt-2 block w-full text-gray-800">
        <option v-for="(level, index) in levels" :key="index" :value="index">
          {{ `${level.name} (${level.exp.toLocaleString()} EXP)` }}
        </option>
      </select>
    </div>

    <label class="mt-2 block">
      <input type="checkbox" v-model="useRatio" />
      <span class="ml-2 text-gray-900">Use proportion</span>
    </label>

    <!-- Materials Ratio -->
    <div v-if="useRatio" class="mt-8">
      <h2 class="text-xl font-semibold text-gray-900">
        Materials Proportion
      </h2>

      <div class="mt-2 flex items-center gap-2">
        <div class="flex-1">
          <input type="number" min="0" v-model="star4Ratio"
            class="block w-full text-gray-900 text-center" />
        </div>
        :
        <div class="flex-1">
          <input type="number" min="0" v-model="star3Ratio"
            class="block w-full text-gray-900 text-center" />
        </div>
        :
        <div class="flex-1">
          <input type="number" min="0" v-model="star2Ratio"
            class="block w-full text-gray-900 text-center" />
        </div>
      </div>
    </div>

    <!-- Materials Needed -->
    <h2 class="mt-8 text-xl font-semibold text-gray-900">Materials Needed</h2>

    <div class="flex gap-4">
      <div class="flex-1">
        <img :src="getImage(0)" :alt="getImageName(0)" :title="getImageName(0)"
          class="p-2" />
        <input type="number" min="0" v-model="star4Need" :disabled="useRatio"
          class="block w-full text-gray-900 text-center" />
      </div>
      <div class="flex-1">
        <img :src="getImage(1)" :alt="getImageName(1)" :title="getImageName(1)"
          class="p-2" />
        <input type="number" min="0" v-model="star3Need" :disabled="useRatio"
          class="block w-full text-gray-900 text-center" />
      </div>
      <div class="flex-1">
        <img :src="getImage(2)" :alt="getImageName(2)" :title="getImageName(2)"
          class="p-2" />
        <input type="number" min="0" v-model="star2Need" :disabled="useRatio"
          class="block w-full text-gray-900 text-center" />
      </div>
    </div>
  </div>
</template>