<template>
  <div class="animate-fade-in relative">
    <div
      class="flex flex-col sm:flex-row justify-between items-start sm:items-center mb-8 bg-primary/5 p-4 rounded-lg border border-primary/20"
    >
      <div>
        <h3 class="font-semibold text-primary">هل بدأت التقديم مسبقاً؟</h3>
        <p class="text-sm text-text-muted">يمكنك العودة لملفك واستكمال المرفقات أو تعديلها.</p>
      </div>
      <AppButton
        @click="isResumeModalOpen = true"
        variant="secondary"
        class="mt-3 sm:mt-0 whitespace-nowrap border-primary text-primary hover:bg-primary/10"
      >
        استكمال طلب سابق
      </AppButton>
    </div>

    <div class="mb-6">
      <h2 class="text-xl font-bold text-text-primary">البيانات الشخصية الأساسية</h2>
      <p class="text-sm text-text-muted mt-1">
        يرجى إدخال بياناتك كما هي مطابقة في الهوية الرسمية.
      </p>
    </div>

    <div
      v-if="applicantStore.error"
      class="mb-6 p-4 bg-danger/10 border-l-4 border-danger rounded text-text-primary"
    >
      <p class="font-semibold text-danger">تنبيه:</p>
      <p class="text-sm">{{ applicantStore.error }}</p>
    </div>

    <form @submit.prevent="submitForm">
      <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
        <AppInput
          id="first_name"
          v-model="form.FirstName"
          label="الاسم الأول"
          required
          placeholder="أدخل الاسم الأول"
        />
        <AppInput
          id="last_name"
          v-model="form.LastName"
          label="اسم العائلة"
          required
          placeholder="أدخل اسم العائلة"
        />
        <AppInput
          id="national_id"
          v-model="form.NationalID"
          label="الرقم الوطني / الإقامة"
          type="number"
          required
          placeholder="أدخل رقم الهوية المكون من 10 أرقام"
        />
        <AppInput
          id="email"
          v-model="form.Email"
          label="البريد الإلكتروني"
          type="email"
          required
          placeholder="example@domain.com"
        />
        <AppInput
          id="phone"
          v-model="form.PhoneNumber"
          label="رقم الهاتف المحمول"
          type="tel"
          required
          placeholder="05xxxxxxxx"
          dir="ltr"
        />
        <CitiesDropdown id="city_id" v-model="form.city_id" label="مدينة الإقامة" required />
      </div>

      <div class="mt-8 flex justify-end">
        <AppButton
          type="submit"
          class="bg-primary hover:bg-primary-dark text-white px-8 py-2 rounded transition-colors"
          :disabled="applicantStore.loading"
        >
          <span v-if="applicantStore.loading">جاري الحفظ...</span>
          <span v-else class="flex items-center">
            حفظ والمتابعة
            <svg
              class="w-5 h-5 mr-2 rtl:rotate-180"
              fill="none"
              stroke="currentColor"
              viewBox="0 0 24 24"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                d="M13 7l5 5m0 0l-5 5m5-5H6"
              ></path>
            </svg>
          </span>
        </AppButton>
      </div>
    </form>

    <ResumeApplicationModal
      :is-open="isResumeModalOpen"
      @close="isResumeModalOpen = false"
      @resumed="handleSuccessfulResume"
    />

    <div
      v-if="showSuccessModal"
      class="fixed inset-0 z-50 flex items-center justify-center bg-black/60 backdrop-blur-sm px-4"
    >
      <div
        class="bg-surface-ground rounded-2xl shadow-2xl max-w-md w-full p-8 text-center animate-fade-in border border-surface-border"
      >
        <div
          class="w-20 h-20 mx-auto bg-green-100 dark:bg-green-900/30 text-green-600 dark:text-green-400 rounded-full flex items-center justify-center mb-6"
        >
          <svg class="w-10 h-10" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="2"
              d="M5 13l4 4L19 7"
            ></path>
          </svg>
        </div>

        <h3 class="text-2xl font-bold text-text-primary mb-2">تم الحفظ بنجاح!</h3>
        <p class="text-text-muted mb-6 leading-relaxed">
          يرجى الاحتفاظ بكود التتبع الخاص بك. ستحتاج إليه إذا أردت استكمال طلبك من جهاز آخر أو
          تعديله لاحقاً.
        </p>

        <div class="bg-surface-section border-2 border-primary/20 rounded-xl p-5 mb-8">
          <p class="text-sm text-text-secondary mb-2 font-medium">كود التتبع السري:</p>
          <p class="text-3xl font-mono font-bold text-primary tracking-widest">
            {{ applicantStore.trackingCode }}
          </p>
        </div>

        <AppButton
          @click="proceedToNextStep"
          class="w-full bg-primary hover:bg-primary-dark text-white py-3 rounded-xl text-lg font-medium transition-colors shadow-lg shadow-primary/30"
        >
          متابعة لرفع المرفقات
        </AppButton>
      </div>
    </div>
  </div>
</template>

<script setup>
import { reactive, ref } from 'vue'
import { useApplicantStore } from '@/stores/applicantStore'
import AppInput from '@/components/ui/AppInput.vue'
import AppButton from '@/components/ui/AppButton.vue'
import CitiesDropdown from '@/components/forms/CitiesDropdown.vue'

// ✅ استيراد نافذة الاستكمال بناءً على المسار الذي ذكرته
import ResumeApplicationModal from '@/views/external-application/steps/ResumeApplicationModal.vue'

const emit = defineEmits(['step-completed'])
const applicantStore = useApplicantStore()

// المتغيرات
const showSuccessModal = ref(false)
const savedApplicantId = ref(null)
const isResumeModalOpen = ref(false) // للتحكم في نافذة الاستكمال

const form = reactive({
  FirstName: '',
  LastName: '',
  NationalID: '',
  Email: '',
  PhoneNumber: '',
  city_id: null,
  ApplicationSource: 'Online',
})

const submitForm = async () => {
  try {
    const payload = {
      ...form,
      city_id: form.city_id ? Number(form.city_id) : null,
    }

    const newApplicant = await applicantStore.createApplicant(payload)

    savedApplicantId.value = newApplicant.id
    showSuccessModal.value = true
  } catch (error) {
    console.error('فشل التسجيل في الخطوة الأولى:', error)
  }
}

const proceedToNextStep = () => {
  showSuccessModal.value = false
  emit('step-completed', { applicantId: savedApplicantId.value })
}

// ✅ دالة التعامل مع النجاح في الاستكمال (القفز للخطوة التالية)
const handleSuccessfulResume = (applicantData) => {
  emit('step-completed', { applicantId: applicantData.id })
}
</script>
