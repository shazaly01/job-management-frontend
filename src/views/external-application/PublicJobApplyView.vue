<template>
  <div class="space-y-8">
    <div v-if="jobRequestStore.loading" class="text-center py-20 flex flex-col items-center gap-4">
      <div class="animate-spin rounded-full h-12 w-12 border-b-2 border-primary"></div>
      <p class="text-text-muted">جاري جلب تفاصيل الوظيفة...</p>
    </div>

    <div
      v-else-if="jobRequestStore.error"
      class="text-center py-20 bg-surface-section rounded-2xl border border-surface-border p-10 space-y-4"
    >
      <svg
        xmlns="http://www.w3.org/2000/svg"
        class="h-20 w-20 text-danger mx-auto"
        fill="none"
        viewBox="0 0 24 24"
        stroke="currentColor"
      >
        <path
          stroke-linecap="round"
          stroke-linejoin="round"
          stroke-width="1.5"
          d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z"
        />
      </svg>
      <h1 class="text-2xl font-bold text-text-primary">عذراً، الوظيفة غير متاحة</h1>
      <p class="text-text-muted max-w-md mx-auto">{{ jobRequestStore.error }}</p>
      <div class="pt-5">
        <AppButton href="/careers" variant="secondary">استعراض كافة الوظائف</AppButton>
      </div>
    </div>

    <div v-else-if="currentJob" class="space-y-8">
      <div
        class="bg-surface-section rounded-2xl border border-surface-border p-6 md:p-8 flex flex-col md:flex-row md:items-center justify-between gap-6 shadow-inner-white"
      >
        <div>
          <span
            class="inline-flex items-center px-3 py-1 rounded-full text-xs font-medium bg-primary/10 text-primary mb-2"
          >
            رقم الطلب: {{ currentJob.RequestNo }}
          </span>
          <h1 class="text-3xl font-bold text-text-primary">{{ currentJob.RequiredMajor }}</h1>
          <p class="text-text-muted mt-2 flex items-center gap-2">
            <svg
              xmlns="http://www.w3.org/2000/svg"
              class="h-5 w-5"
              fill="none"
              viewBox="0 0 24 24"
              stroke="currentColor"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                d="M19 21V5a2 2 0 00-2-2H7a2 2 0 00-2 2v16m14 0h2m-2 0h-5m-9 0H3m2 0h5M9 7h1m-1 4h1m4-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"
              />
            </svg>
            الإدارة: {{ currentJob.Department?.Name || 'غير محددة' }}
          </p>
        </div>
        <AppButton @click="scrollToForm" variant="primary" size="lg">قدّم الآن</AppButton>
      </div>

      <div class="grid grid-cols-1 lg:grid-cols-3 gap-8">
        <div class="lg:col-span-1 space-y-6">
          <div class="bg-surface-section p-6 rounded-2xl border border-surface-border space-y-4">
            <h2 class="text-lg font-bold text-text-primary border-b border-surface-border pb-3">
              متطلبات الوظيفة
            </h2>

            <div class="flex items-center gap-3">
              <span class="p-2 bg-primary/10 text-primary rounded-lg">🎓</span>
              <div>
                <p class="text-xs text-text-muted">المؤهل المطلوب</p>
                <p class="font-medium text-text-primary">{{ currentJob.RequiredDegreeLevel }}</p>
              </div>
            </div>

            <div class="flex items-center gap-3">
              <span class="p-2 bg-primary/10 text-primary rounded-lg">💼</span>
              <div>
                <p class="text-xs text-text-muted">سنوات الخبرة</p>
                <p class="font-medium text-text-primary">
                  {{ currentJob.RequiredYearsOfExperience }} سنوات على الأقل
                </p>
              </div>
            </div>
          </div>

          <div class="bg-surface-section p-6 rounded-2xl border border-surface-border">
            <h2
              class="text-lg font-bold text-text-primary border-b border-surface-border pb-3 mb-4"
            >
              وصف الوظيفة والمسؤوليات
            </h2>
            <div
              class="prose prose-sm prose-primary max-w-none text-text-secondary whitespace-pre-line"
            >
              {{ currentJob.JobDescription || 'لا يوجد وصف تفصيلي متاح لهذه الوظيفة حالياً.' }}
            </div>
          </div>
        </div>

        <div
          id="application-form"
          class="lg:col-span-2 bg-surface-section p-6 md:p-8 rounded-2xl border border-surface-border"
        >
          <h2 class="text-2xl font-bold text-text-primary mb-2">نموذج طلب التوظيف</h2>
          <p class="text-text-muted mb-8 border-b border-surface-border pb-5">
            يرجى تعبئة كافة الحقول المطلوبة لضمان مراجعة طلبك بأسرع وقت.
          </p>

          <PublicApplicationForm
            :fixed-job-request-id="currentJob.id"
            :fixed-job-title="currentJob.RequiredMajor"
          />
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { onMounted, computed } from 'vue'
import { useRoute } from 'vue-router'
import { useJobRequestStore } from '@/stores/jobRequestStore'
import AppButton from '@/components/ui/AppButton.vue'
import PublicApplicationForm from '@/views/external-application/ApplicationWizard.vue' // ✅ استيراد النموذج العام

const route = useRoute()
const jobRequestStore = useJobRequestStore()

// جلب الوظيفة الحالية من الستور
const currentJob = computed(() => jobRequestStore.currentJobRequest)

// عند تحميل الصفحة، نأخذ الـ slug من الرابط ونجلب البيانات
onMounted(async () => {
  const slug = route.params.slug
  if (slug) {
    await jobRequestStore.fetchJobRequestBySlug(slug)
  }
})

// دالة للانتقال السلس للنموذج عند الضغط على "قدم الآن"
const scrollToForm = () => {
  document.getElementById('application-form')?.scrollIntoView({ behavior: 'smooth' })
}
</script>
