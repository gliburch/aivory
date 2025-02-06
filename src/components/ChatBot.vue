<script setup>
import { ref, onMounted, computed } from 'vue'
import axios from 'axios'
import { marked } from 'marked'
import xss from 'xss'
import AnimatedLoading from '@/components/AnimatedLoading.vue'

const API_BASE_URL = 'https://aivory.vercel.app'

const MESSAGE_EXAMPLES = [
  {
    title: '간단한 인사를 해보자',
    message: '안녕 👋',
  },
  {
    title: '자기 소개를 시켜보자',
    message: '너에 대해 알고싶어.',
  },
  {
    title: '내 이름을 물어보게 해보자',
    message: '내 이름을 물어봐줄래?\n그리고 앞으로는 그 이름으로 불러줘.',
  },
  {
    title: '$HUB가 뭔지 물어보자',
    message: '$HUB에 대해 간단히 설명해줄래?',
  },
  {
    title: 'Aivory의 미래를 물어보자',
    message: 'Aivory는 앞으로 어떻게 발전할거 같아?',
  },
]

const { threadId: threadId = '', latestMessageCounts: initialMessageCounts = 100 } = defineProps({
  threadId: String,
  latestMessageCounts: Number,
})

const isLoading = ref(true)
const messages = ref([])
const newMessage = ref('')
const messagesContainer = ref(null)
const currentThreadId = ref(threadId)

// localStorage에서 시도한 메시지 목록을 가져옴
const messageExamplesTried = ref(
  JSON.parse(localStorage.getItem('aivory:assistant:usedMessageExamples') || '[]'),
)

// 시도하지 않은 메시지 예시만 필터링하는 computed 속성
const availableMessageExamples = computed(() => {
  return MESSAGE_EXAMPLES.filter((example) => !messageExamplesTried.value.includes(example.title))
})

// Sort messages by created_at in descending order
const sortedMessages = computed(() => {
  return [...messages.value].sort((a, b) => new Date(a.created_at) - new Date(b.created_at))
})

// Scroll to bottom function
const scrollToBottom = () => {
  if (messagesContainer.value) {
    messagesContainer.value.scrollTop = messagesContainer.value.scrollHeight
  }
}

const fetchMessages = async (limit = initialMessageCounts) => {
  try {
    const response = await axios.get(
      `${API_BASE_URL}/api/assistants/threads/${currentThreadId.value}/messages`,
      { params: { limit } },
    )
    return response.data.data
  } catch (error) {
    console.error('Error fetching messages:', error)
    return []
  }
}

const fetchLastMessage = async () => {
  try {
    const response = await axios.get(
      `${API_BASE_URL}/api/assistants/threads/${currentThreadId.value}/messages`,
      { params: { limit: 1 } },
    )
    const lastMessage = response.data.data[0]
    return lastMessage?.role === 'assistant' ? lastMessage : null
  } catch (error) {
    console.error('Error fetching last message:', error)
    return null
  }
}

const addMessage = async (content, role = 'user') => {
  messages.value.push({
    id: `temp_${Date.now()}`,
    content: [{ type: 'text', text: { value: content } }],
    role: role,
    created_at: new Date().toISOString(),
  })
  return messages.value[messages.value.length - 1] // 추가된 메시지 반환
}

const markMessageExampleAsUsed = (messageContent) => {
  const example = MESSAGE_EXAMPLES.find((ex) => ex.message === messageContent)
  if (example && !messageExamplesTried.value.includes(example.title)) {
    messageExamplesTried.value.push(example.title)
    localStorage.setItem(
      'aivory:assistant:usedMessageExamples',
      JSON.stringify(messageExamplesTried.value),
    )
  }
}

const dismissMessageExample = (title) => {
  if (!messageExamplesTried.value.includes(title)) {
    messageExamplesTried.value.push(title)
    localStorage.setItem(
      'aivory:assistant:usedMessageExamples',
      JSON.stringify(messageExamplesTried.value),
    )
  }
}

const sendMessage = async (messageContent) => {
  if (!messageContent?.trim()) return

  markMessageExampleAsUsed(messageContent)
  isLoading.value = true
  await addMessage(messageContent)
  scrollToBottom()

  try {
    if (!currentThreadId.value) {
      // 새 쓰레드 생성
      const response = await axios.post(`${API_BASE_URL}/api/assistants/threads`)
      const { threadId } = response.data
      currentThreadId.value = threadId
      window.localStorage.setItem('aivory:assistant:userThreadId', threadId)
    }

    await axios.post(`${API_BASE_URL}/api/assistants/threads/${currentThreadId.value}/messages`, {
      content: messageContent,
    })
    const assistantMessage = await fetchLastMessage()
    if (assistantMessage) {
      await addMessage(assistantMessage.content[0].text.value, 'assistant')
      setTimeout(() => {
        scrollToBottom()
      }, 0)
    }
  } catch (error) {
    console.error('Error sending message:', error)
    messages.value.pop()
  } finally {
    isLoading.value = false
  }
}

const handleSubmit = async (e) => {
  e.preventDefault()
  if (!newMessage.value.trim()) return

  const messageToSend = newMessage.value
  newMessage.value = ''
  await sendMessage(messageToSend)
}

// Configure marked options
marked.setOptions({
  breaks: true, // Enable line breaks
  gfm: true, // Enable GitHub Flavored Markdown
})

const useMessageFormatting = () => {
  const formatMessageContent = (message) => {
    if (!message.content || !Array.isArray(message.content)) return ''

    const content = message.content
      .filter((content) => content.type === 'text')
      .map((content) => content.text.value)
      .join('\n')

    // Convert markdown to HTML, then sanitize
    const htmlContent = marked(content)
    return xss(htmlContent, {
      whiteList: {
        p: [],
        b: [],
        i: [],
        em: [],
        strong: [],
        code: [],
        pre: [],
        br: [],
        ul: [],
        ol: [],
        li: [],
      },
    })
  }

  return {
    formatMessageContent,
  }
}

const { formatMessageContent } = useMessageFormatting()

onMounted(async () => {
  if (currentThreadId.value) {
    const initialMessages = await fetchMessages(initialMessageCounts)
    messages.value = initialMessages
  }
  isLoading.value = false
  setTimeout(() => {
    scrollToBottom()
  }, 0)
})
</script>

<template>
  <div class="flex flex-col">
    <!-- Messages container with ref -->
    <div ref="messagesContainer" class="flex-1 overflow-auto">
      <div class="flex flex-col gap-y-4 p-5 pt-15">
        <template v-for="message in sortedMessages" :key="message.id">
          <!-- Receiver message -->
          <div v-if="message.role === 'assistant'" class="flex items-start">
            <div class="relative bg-white rounded-2xl px-3 py-2 max-w-[70%]">
              <figure>
                <img
                  src="@/assets/images/fig-vory-1.jpeg"
                  class="absolute -top-8.5 left-2.5 w-10 h-10 object-cover rounded-full shadow-sm"
                  alt=""
                />
              </figure>
              <div class="markdown text-primary" v-html="formatMessageContent(message)"></div>
            </div>
          </div>
          <!-- Sender message -->
          <div v-else class="flex items-start justify-end">
            <div class="bg-primary rounded-2xl px-3 py-2 max-w-[70%]">
              <div class="markdown text-white" v-html="formatMessageContent(message)"></div>
            </div>
          </div>
        </template>
        <!-- Message examples -->
        <template v-if="false">
        <!-- <template v-if="!isLoading"> -->
          <div
            v-for="{ title, message } in availableMessageExamples"
            :key="title"
            class="flex items-start justify-end"
          >
            <div
              class="relative flex items-stretch border-2 border-primary border-dashed rounded-2xl overflow-hidden transition-all group hover:-translate-x-1.5"
            >
              <button
                type="button"
                class="flex-1 pr-10 py-1.5 pl-3 cursor-pointer transition-colors hover:bg-primary/10"
                @click="sendMessage(message)"
              >
                {{ title }}
              </button>
              <button
                type="button"
                class="absolute top-[50%] right-1 translate-y-[-50%] w-8 h-8 flex items-center justify-center rounded-full bg-transparent hover:bg-primary/10 text-md cursor-pointer"
                @click="dismissMessageExample(title)"
              >
                X
              </button>
            </div>
          </div>
        </template>
      </div>
    </div>

    <!-- Input area -->
    <div class="border-t border-primary/15 p-5">
      <form @submit="handleSubmit" class="flex gap-2">
        <input
          v-model="newMessage"
          type="text"
          :placeholder="isLoading ? '답을 기다리는 중이예요...' : '뭐든 이야기해 봐요 :)'"
          class="w-[80%] rounded-full border border-primary px-4 py-2 text-primary focus:outline-none focus:bg-white/40"
        />
        <button
          type="submit"
          class="flex-1 min-w-20 bg-primary text-white text-nowrap px-4 py-2 rounded-full cursor-pointer hover:opacity-80 disabled:opacity-50 disabled:cursor-not-allowed"
          :disabled="isLoading"
        >
          <AnimatedLoading v-if="isLoading" color="var(--color-white)" :size="16" class="mx-auto" />
          <template v-else>보내기</template>
        </button>
      </form>
    </div>
  </div>
</template>

<style scoped>
.markdown {
  font-size: 1rem;
  line-height: 1.6;
}
.markdown:deep(> * + *) {
  margin-top: 0.75em;
}
.markdown:deep(p) {
  white-space: pre-wrap;
}
.markdown:deep(code) {
  padding: 0.25rem 0.5rem;
  background-color: rgba(0, 0, 0, 0.1);
  border-radius: 0.25rem;
  word-break: break-all;
  word-wrap: break-word;
}
.markdown:deep(pre) {
  padding: 0.75rem;
  background-color: rgba(0, 0, 0, 0.1);
  border-radius: 0.5rem;
  overflow-x: auto;
}
.markdown:deep(pre code) {
  background-color: transparent;
  padding: 0;
}
.markdown:deep(ul),
.markdown:deep(ol) {
  padding-left: 1.5rem;
}
.markdown:deep(ul) {
  list-style-type: disc;
}
.markdown:deep(ol) {
  list-style-type: decimal;
}
.markdown:deep(strong) {
  font-weight: 700;
}
.markdown:deep(em) {
  font-style: italic;
}
</style>
