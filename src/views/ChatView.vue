<script setup>
import ChatBot from '@/components/ChatBot.vue'
// import ChatBotLegacy from '@/components/ChatBotLegacy.vue'
</script>

<template>
  <section
    :class="[
      'relative w-full h-auto mx-auto bg-white/60 rounded-3xl shadow-2xl border-2 border-white backdrop-blur-md text-primary',
      'lg:w-[500px] lg:h-[650px]',
    ]"
  >
    <h1
      class="z-1 absolute top-[-20px] left-[20px] py-1 px-3 bg-secondary text-white rounded-lg text-base"
    >
      Aivory ❤️
    </h1>
    <ChatBot threadId="thread_aohLbzFLIFj68pQVYMMqNYoX" class="h-full" />
  </section>
</template>
