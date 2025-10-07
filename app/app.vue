<template>
  <div dir="rtl">
    <div class="w-full flex items-center h-screen">
      <div class="app-container flex flex-col bg-cover h-screen px-4 pb-4">
        <div
          class="alert-box w-80 h-max alert-box text-white mt-6 flex items-center justify-center rounded-3xl flex-col border-nutral-850 w-full px-4 pt-4 pb-8"
        >
          <div class="w-12 h-12">
            <icon-svg name="icons:icon" class="text-5xl" />
          </div>
          <span class="mt-4 text-white text-sm font-medium"
            >سلام خوش‌آمدی
          </span>
          <span class="text-sm text-justify text-neutral-064 font-medium">
            من دستیار رویدادم و میتونم بهت بگم شرکت کنندگان چی پرسیدن.
            اگه سوالی راجب به رویداد داری هم در خدمتم</span
          >
        </div>

        <MessageBox :messages="messages" :loading="loading" />
        <ChatInput v-model="userInput" @send-message="sendMessage" />
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { ref } from 'vue';
import MessageBox from './components/MessageBox.vue';
import ChatInput from './components/ChatInput.vue';

// 1. نوع ID برای سازگاری با کدهای قبلی اصلاح شد
interface Message {
  id: number | string;
  text: string;
  type: 'sender' | 'receiver';
}

export default {
  components: {
    MessageBox,
    ChatInput
  },
  setup() {
    const userInput = ref('');
    const messages = ref<Message[]>([]);
    const loading = ref(false);

    // 2. آدرس بک‌اند شما
    const backendUrl = 'http://89.251.9.191:8000';

    // 3. تابع sendMessage برای ارسال درخواست به سرور
    const sendMessage = async () => {
      if (userInput.value.trim()) {
        const presenterQuestion = userInput.value;
        
        // نمایش فوری پیام مجری در رابط کاربری
        messages.value.push({
          id: 'msg-' + Date.now(),
          text: presenterQuestion,
          type: 'sender'
        });
        
        userInput.value = '';
        loading.value = true;

        try {
          // ارسال درخواست به endpoint مخصوص مجری
          const response = await fetch(`${backendUrl}/api/v1/presenter/chat`, {
            method: 'POST',
            headers: {
              'Content-Type': 'application/json',
            },
            // بدنه درخواست فقط شامل 'question' است
            body: JSON.stringify({
              question: presenterQuestion
            })
          });

          if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
          }

          const data = await response.json();
          
          // نمایش پاسخ تحلیلی ربات
          messages.value.push({
            id: 'res-' + Date.now(),
            text: data.answer, // فیلد پاسخ 'answer' است
            type: 'receiver'
          });

        } catch (error) {
          console.error('Error sending presenter question:', error);
          messages.value.push({
            id: 'error-' + Date.now(),
            text: 'خطا در تحلیل داده‌ها. لطفاً اتصال سرور را بررسی کنید.',
            type: 'receiver'
          });
        } finally {
          loading.value = false;
        }
      }
    };
    
    return {
      userInput,
      messages,
      loading,
      sendMessage
    };
  }
};
</script>

<style>
/* استایل‌های قبلی شما */
@import "../assets/css/main.css";
.app-container {
  width: min(100vw, 360px);
  margin: 0 auto;
  background: #000 url("/bg.svg") center no-repeat;
}
.alert-box {
  background-color: #00000014;
  backdrop-filter: blur(2px);
  border: 1px solid #cacaca;
}
.text-neutral-064 {
  color: rgba(255, 255, 255, 0.64);
}
.text-neutral-500 {
  color: rgba(158, 158, 158, 1);
}
.text-neutral-0 {
  color: rgba(255, 255, 255, 1);
}
.text-nutral-50{
  color: rgba(158, 158, 158, 1);
}
.border-nutral-850{
  border-color: rgba(47, 47, 47, 1);
}
.bg-primary-500{
  color:rgba(0, 110, 213, 1);
}
.message-box {
  height: calc(100vh - 298px);
}
.textarea {
  resize: none;
}
.textarea::placeholder {
  padding-top: 0.625rem;
  font-size: 14px;
}
</style>