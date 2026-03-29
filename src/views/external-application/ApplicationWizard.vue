<template>
  <AppCard class="max-w-5xl mx-auto mt-10 p-6 shadow-lg bg-surface-ground">
    <div class="text-center mb-10">
      <h1 class="text-3xl font-bold text-text-primary">بوابة التوظيف</h1>
      <p class="text-text-muted mt-2">تقديم طلب توظيف جديد - يرجى إكمال الخطوات التالية</p>
    </div>

    <AppStepper :steps="stepNames" :currentStep="currentStep" />

    <div class="mt-8 transition-all duration-300">
      <component
        :is="currentStepComponent"
        :applicant-id="applicantId"
        :fixed-job-request-id="props.fixedJobRequestId"
        :fixed-job-title="props.fixedJobTitle"
        @step-completed="handleStepCompleted"
        @go-back="prevStep"
      />
    </div>
  </AppCard>
</template>

<script setup>
import { ref, markRaw, computed } from 'vue'
import AppCard from '@/components/ui/AppCard.vue'
import AppStepper from '@/components/ui/AppStepper.vue'

import Step1_BasicInfo from './steps/Step1_BasicInfo.vue'
import Step2_Qualifications from './steps/Step2_Qualifications.vue'
import Step3_Experiences from './steps/Step3_Experiences.vue'
import Step4_Skills from './steps/Step4_Skills.vue'
import Step5_Documents from './steps/Step5_Documents.vue'
import Step6_ReviewSubmit from './steps/Step6_ReviewSubmit.vue'

// ✅ استقبال البيانات من الشاشة الخارجية (إن وجدت)
const props = defineProps({
  fixedJobRequestId: {
    type: [Number, String],
    default: null,
  },
  fixedJobTitle: {
    type: String,
    default: '',
  },
})

const currentStep = ref(1)
const applicantId = ref(null)

const stepNames = [
  'البيانات الأساسية',
  'المؤهلات العلمية',
  'الخبرات المهنية',
  'المهارات',
  'المرفقات',
  'التقديم',
]

const components = [
  markRaw(Step1_BasicInfo),
  markRaw(Step2_Qualifications),
  markRaw(Step3_Experiences),
  markRaw(Step4_Skills),
  markRaw(Step5_Documents),
  markRaw(Step6_ReviewSubmit),
]

const currentStepComponent = computed(() => components[currentStep.value - 1])

const handleStepCompleted = (payload) => {
  if (currentStep.value === 1 && payload?.applicantId) {
    applicantId.value = payload.applicantId
  }

  if (currentStep.value < components.length) {
    currentStep.value++
    window.scrollTo({ top: 0, behavior: 'smooth' })
  }
}

const prevStep = () => {
  if (currentStep.value > 1) {
    currentStep.value--
    window.scrollTo({ top: 0, behavior: 'smooth' })
  }
}
</script>
