<script setup>
import { ref, onMounted, computed } from 'vue'
import axios from 'axios'
import { marked } from 'marked'
import xss from 'xss'
import AnimatedLoading from '@/components/AnimatedLoading.vue'

const { threadId = '' } = defineProps({
  threadId: String,
})

const isLoading = ref(true)
const messages = ref([])
const newMessage = ref('')
const messagesContainer = ref(null)
const currentThreadId = ref(threadId)

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

const fetchMessages = async (limit = 10) => {
  try {
    const response = await axios.get(
      `http://localhost:3000/api/assistants/threads/${currentThreadId.value}/messages`,
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
      `http://localhost:3000/api/assistants/threads/${currentThreadId.value}/messages`,
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

const sendMessage = async (messageContent) => {
  if (!messageContent?.trim()) return

  isLoading.value = true
  await addMessage(messageContent)
  scrollToBottom()

  try {
    if (!currentThreadId.value) {
      // 새 쓰레드 생성
      const response = await axios.post('http://localhost:3000/api/assistants/threads')
      const { threadId } = response.data
      currentThreadId.value = threadId
      window.localStorage.setItem('aivory:assistant:userThreadId', threadId)
    }

    await axios.post(
      `http://localhost:3000/api/assistants/threads/${currentThreadId.value}/messages`,
      {
        content: messageContent,
      },
    )
    const assistantMessage = await fetchLastMessage()
    if (assistantMessage) {
      await addMessage(assistantMessage.content[0].text.value, 'assistant')
      scrollToBottom()
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
    const initialMessages = await fetchMessages()
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
      <div class="flex flex-col gap-y-4 p-5 pt-10">
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
      </div>
    </div>

    <!-- Input area -->
    <div class="border-t border-primary/15 p-5">
      <form @submit="handleSubmit" class="flex gap-2">
        <input
          v-model="newMessage"
          type="text"
          :placeholder="isLoading ? '답을 기다리는중!' : '뭐든 말해봐!'"
          class="flex-1 rounded-full border border-primary px-4 py-2 text-primary focus:outline-none focus:bg-white/40"
        />
        <button
          type="submit"
          class="min-w-20 bg-primary text-white text-nowrap px-4 py-2 rounded-full cursor-pointer hover:opacity-80 disabled:opacity-50 disabled:cursor-not-allowed"
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
