<template>
  <div class="flex items-center justify-between w-full mb-8 relative">
    <div
      class="absolute left-0 top-1/2 transform -translate-y-1/2 w-full h-1 bg-surface-section z-0 rounded-full"
    ></div>

    <div
      class="absolute left-0 top-1/2 transform -translate-y-1/2 h-1 bg-primary transition-all duration-300 z-0 rounded-full"
      :style="{ width: progressWidth }"
    ></div>

    <div
      v-for="(step, index) in steps"
      :key="index"
      class="relative z-10 flex flex-col items-center"
    >
      <div
        class="w-10 h-10 rounded-full flex items-center justify-center text-sm font-bold border-4 transition-colors duration-300"
        :class="[
          currentStep > index + 1 ? 'bg-primary border-primary text-text-primary' : '',
          currentStep === index + 1 ? 'bg-surface-ground border-primary text-primary' : '',
          currentStep < index + 1 ? 'bg-surface-section border-surface-border text-text-muted' : '',
        ]"
      >
        <svg
          v-if="currentStep > index + 1"
          class="w-5 h-5 text-white"
          fill="none"
          stroke="currentColor"
          viewBox="0 0 24 24"
        >
          <path
            stroke-linecap="round"
            stroke-linejoin="round"
            stroke-width="2"
            d="M5 13l4 4L19 7"
          ></path>
        </svg>
        <span v-else>{{ index + 1 }}</span>
      </div>

      <span
        class="mt-2 text-xs font-semibold whitespace-nowrap transition-colors duration-300"
        :class="currentStep >= index + 1 ? 'text-text-primary' : 'text-text-muted'"
      >
        {{ step }}
      </span>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  steps: {
    type: Array,
    required: true,
  },
  currentStep: {
    type: Number,
    required: true,
  },
})

// حساب عرض خط التقدم الملون بناءً على الخطوة الحالية
const progressWidth = computed(() => {
  const total = props.steps.length - 1
  const current = props.currentStep - 1
  return `${(current / total) * 100}%`
})
</script>
