<script setup>
import { RouterLink, RouterView } from 'vue-router'
import AivoryHero from '@/components/AivoryHero.vue'
</script>

<template>
  <div
    :class="[
      'max-w-[800px] mx-auto',
      'lg:grid lg:grid-cols-2 lg:max-w-[1920px] lg:items-center',
    ]"
  >
    <header :class="['lg:overflow-hidden lg:w-[500px] lg:rounded-3xl lg:shadow-2xl lg:mx-auto']">
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
      <AivoryHero></AivoryHero>
      <nav
        :class="[
          'fixed bottom-0 left-1/2 -translate-x-1/2 w-full p-4',
          'bg-white/40 rounded-t-[40px] backdrop-blur-md flex justify-center space-x-4',
          'lg:static lg:translate-none lg:rounded-none',
        ]"
      >
        <a
          href="https://quickswap.exchange/#/swap?currency0=ETH&currency1=0x634d198Ec69b87F24901574C388fe8f90ADf2B50&swapIndex=2"
          target="_blank"
          class="leading-[40px] px-3 py-2 rounded-tl-[30px] rounded-br-[30px] rounded-tr-[15px] rounded-bl-[15px] bg-primary text-white text-lg shadow-md"
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
          class="leading-[40px] px-3 py-2 rounded-tl-[30px] rounded-br-[30px] rounded-tr-[15px] rounded-bl-[15px] bg-secondary text-white text-lg shadow-md"
        >
          Aivory 토큰
        </a>
      </nav>
    </header>
    <main class="lg:h-screen lg:overflow-auto">
      <RouterView />
    </main>
  </div>
</template>
