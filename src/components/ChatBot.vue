<script setup>
import { ref, onMounted, computed } from 'vue'
import axios from 'axios'
import { marked } from 'marked'
import xss from 'xss'

const isLoading = ref(true)
const messages = ref([])
const newMessage = ref('')
const messagesContainer = ref(null)

// Sort messages by created_at in descending order
const sortedMessages = computed(() => {
  return [...messages.value].sort((a, b) => 
    new Date(a.created_at) - new Date(b.created_at)
  )
})

// Scroll to bottom function
const scrollToBottom = () => {
  if (messagesContainer.value) {
    messagesContainer.value.scrollTop = messagesContainer.value.scrollHeight
  }
}

const fetchMessages = async () => {
  try {
    const response = await axios.get('http://localhost:3000/api/assistants/threads/thread_aohLbzFLIFj68pQVYMMqNYoX/messages?limit=100')
    messages.value = response.data.data
  } catch (error) {
    console.error('Error fetching messages:', error)
  } finally {
    isLoading.value = false
  }
}

const handleSubmit = async (e) => {
  e.preventDefault()
  if (!newMessage.value.trim()) return

  const messageToSend = newMessage.value
  newMessage.value = ''
  isLoading.value = true
  try {
    await axios.post('http://localhost:3000/api/assistants/threads/thread_aohLbzFLIFj68pQVYMMqNYoX/messages', {
      content: messageToSend
    })
    await fetchMessages()
    scrollToBottom()
  } catch (error) {
    console.error('Error sending message:', error)
  } finally {
    isLoading.value = false
  }
}

// Configure marked options
marked.setOptions({
  breaks: true, // Enable line breaks
  gfm: true,    // Enable GitHub Flavored Markdown
})

const useMessageFormatting = () => {
  const formatMessageContent = (message) => {
    if (!message.content || !Array.isArray(message.content)) return ''
    
    const content = message.content
      .filter(content => content.type === 'text')
      .map(content => content.text.value)
      .join('\n')
      
    // Convert markdown to HTML, then sanitize
    const htmlContent = marked(content)
    return xss(htmlContent, {
      whiteList: {
        p: [], b: [], i: [], em: [], strong: [], 
        code: [], pre: [], br: [], ul: [], ol: [], li: []
      }
    })
  }

  return {
    formatMessageContent
  }
}

const { formatMessageContent } = useMessageFormatting()

onMounted(async () => {
  await fetchMessages()
  scrollToBottom()
})
</script>

<template>
  <div class="flex flex-col h-screen">
    <!-- Messages container with ref -->
    <div 
      ref="messagesContainer"
      class="flex-1 overflow-y-auto p-4 space-y-4"
    >
      <template v-for="message in sortedMessages" :key="message.id">
        <!-- Receiver message -->
        <div v-if="message.role === 'assistant'" class="flex items-start">
          <div class="bg-gray-100 rounded-lg p-3 max-w-[70%]">
            <div class="markdown text-gray-800 [&>*]:mb-3 [&>*:last-child]:mb-0" v-html="formatMessageContent(message)"></div>
          </div>
        </div>

        <!-- Sender message -->
        <div v-else class="flex items-start justify-end">
          <div class="bg-[var(--vt-c-highlight)] rounded-lg p-3 max-w-[70%]">
            <div class="markdown text-white [&>*]:mb-3 [&>*:last-child]:mb-0" v-html="formatMessageContent(message)"></div>
          </div>
        </div>
      </template>
    </div>

    <!-- Input area -->
    <div class="border-t p-4">
      <form @submit="handleSubmit" class="flex gap-2">
        <input
          v-model="newMessage"
          type="text"
          placeholder="메시지 입력"
          class="flex-1 rounded-full border border-gray-300 px-4 py-2 focus:outline-none focus:border-[var(--vt-c-highlight)]"
        />
        <button
          type="submit"
          class="bg-[var(--vt-c-highlight)] text-white px-6 py-2 rounded-full cursor-pointer hover:opacity-80 disabled:opacity-50 disabled:cursor-not-allowed transition-colors"
          :disabled="isLoading"
        >
          보내기
        </button>
      </form>
    </div>
  </div>
</template>

<style>
.markdown {
  @apply text-base leading-relaxed;
}
.markdown p {
  @apply whitespace-pre-wrap;
}
.markdown code {
  @apply px-1 py-0.5 bg-black/10 rounded;
}
.markdown pre {
  @apply p-3 bg-black/10 rounded-lg overflow-x-auto;
}
.markdown pre code {
  @apply bg-transparent p-0;
}
.markdown ul, .markdown ol {
  @apply pl-6;
}
.markdown ul {
  @apply list-disc;
}
.markdown ol {
  @apply list-decimal;
}
.markdown strong {
  @apply font-bold;
}
.markdown em {
  @apply italic;
}
</style>
