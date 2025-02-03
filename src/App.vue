<script setup>
import { ref, Transition } from 'vue'
import { RouterLink, RouterView } from 'vue-router'
import { Typed } from '@duskmoon/vue3-typed-js'
import AivoryHero from '@/components/AivoryHero.vue'
import ChatBot from '@/components/ChatBot.vue'

const { userThreadId = localStorage.getItem('aivory:assistant:userThreadId') || '' } = defineProps({
  userThreadId: String,
})

const isTalkingInline = ref(false)
</script>

<template>
  <div
    :class="[
      'max-w-[800px] mx-auto',
      'lg:grid lg:grid-cols-2 lg:max-w-[1200px] lg:h-screen lg:items-center',
    ]"
  >
    <header class="lg:overflow-hidden lg:w-[500px] lg:mx-auto lg:rounded-3xl lg:shadow-2xl">
      <nav class="bg-primary text-white px-5 py-4 flex justify-between items-center">
        <h1 class="text-2xl font-bold">
          <RouterLink :to="{ name: 'home' }">Aivory</RouterLink>
        </h1>
        <div class="flex items-center space-x-4">
          <a
            href="https://vaulted-pencil-b8c.notion.site/hub-174dc2175f7d8006a52dfafc345d6dd4"
            target="_blank"
            class="text-white hover:underline"
            >$HUB</a
          >
          <a
            href="https://vaulted-pencil-b8c.notion.site/56-Aivory-16ddc2175f7d80038333fc5e58d2f99d"
            target="_blank"
            class="text-white hover:underline"
            >Future</a
          >
          <a
            href="https://vaulted-pencil-b8c.notion.site/58-Aivory-16ddc2175f7d807bbafee860c7d162b7?pvs=4"
            target="_blank"
            class="text-white hover:underline"
            >White Paper</a
          >
          <!-- <RouterLink
            :to="{name: 'chat'}"
            class="text-white"
            >Chat
          </RouterLink> -->
          <a href="https://t.me/iamaivory" target="_blank">
            <img
              src="@/assets/images/icon-telegram.svg"
              alt="Telegram"
              class="w-6 h-6 inline-block"
            />
          </a>
        </div>
      </nav>
      <AivoryHero />
      <div class="mt-[-30%] mb-[30%] lg:relative lg:mt-auto lg:mb-auto">
        <section
          :class="[
            'relative w-[340px] mx-auto bg-white/60 rounded-xl shadow-lg border-2 border-white backdrop-blur-md text-primary',
            'lg:absolute lg:bottom-15 lg:left-[50%] lg:translate-x-[-50%] lg:transition-transform lg:hover:-translate-y-0.5',
          ]"
        >
          <h1
            class="absolute top-[-20px] left-[20px] py-1 px-3 bg-secondary text-white rounded-lg text-base"
          >
            Aivory ❤️
          </h1>
          <Transition name="fade" mode="out-in">
            <div v-if="!isTalkingInline" class="min-h-23 p-5">
              <Typed
                :options="{
                  strings: [
                    '반가워요 저는 에이아이보리 라고 해요 :)&nbsp;<br>( ... 대화하려면 클릭해요 )',
                  ],
                  typeSpeed: 35,
                  startDelay: 1000,
                  cursorChar: '_',
                }"
              >
                <span class="typing inline text-md"></span>
              </Typed>
              <button
                type="button"
                class="block lg:hidden overflow-hidden absolute top-0 right-0 bottom-0 left-0 border-none bg-transparent text-[0px] -indent-[999px] cursor-pointer"
                @click="isTalkingInline = true"
              >
                바로 대화해보기
              </button>
              <RouterLink
                class="hidden lg:block overflow-hidden absolute top-0 right-0 bottom-0 left-0 text-[0px] -indent-[999px]"
                :to="{ name: 'chat' }"
              >
                대화 메뉴로 가기
              </RouterLink>
            </div>
            <div v-else>
              <ChatBot :threadId="userThreadId" :latestMessageCounts="3" />
            </div>
          </Transition>
        </section>
      </div>
      <nav
        v-if="!isTalkingInline"
        :class="[
          'fixed bottom-0 left-1/2 -translate-x-1/2 w-full p-4',
          'bg-white/40 rounded-t-[40px] backdrop-blur-md flex justify-center space-x-4',
          'lg:static lg:translate-none lg:rounded-none',
        ]"
      >
        <a
          href="https://quickswap.exchange/#/swap?currency0=ETH&currency1=0x634d198Ec69b87F24901574C388fe8f90ADf2B50&swapIndex=2"
          target="_blank"
          class="leading-[40px] px-3 py-2 rounded-tl-[30px] rounded-br-[30px] rounded-tr-[15px] rounded-bl-[15px] bg-primary hover:bg-primary/85 text-white text-lg shadow-md"
        >
          <img
            src="@/assets/images/icon-polygon.svg"
            alt=""
            width="45"
            height="45"
            class="inline-block"
          />
          폴리곤으로 구매 (POL)
        </a>
        <a
          href="https://polygonscan.com/token/0x634d198Ec69b87F24901574C388fe8f90ADf2B50"
          target="_blank"
          class="leading-[40px] px-3 py-2 rounded-tl-[30px] rounded-br-[30px] rounded-tr-[15px] rounded-bl-[15px] bg-secondary hover:bg-secondary/85 text-white text-lg shadow-md"
        >
          Aivory 토큰
        </a>
      </nav>
    </header>
    <main>
      <RouterView />
    </main>
  </div>
</template>
