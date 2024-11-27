
<template>
  <div class="p-4 max-w-4xl mx-auto">
    <div class="flex space-x-2 mb-4">
      <button @click="handleBackClick" class="p-2 bg-gray-200 rounded hover:bg-gray-300" title="Назад">
        <Undo2 class="w-5 h-5" />
      </button>
      <button @click="handleForwardClick" class="p-2 bg-gray-200 rounded hover:bg-gray-300" title="Вперед">
        <Redo2 class="w-5 h-5" />
      </button>
      <button @click="insertHeader" class="p-2 bg-gray-200 rounded hover:bg-gray-300" title="Заголовок">
        <Heading2 class="w-5 h-5" />
      </button>
      <button @click="convertToParagraph" class="p-2 bg-gray-200 rounded hover:bg-gray-300" title="Абзац">
        <Type class="w-5 h-5" />
      </button>
      <button @click="triggerImageUpload" class="p-2 bg-gray-200 rounded hover:bg-gray-300" title="Загрузить изображение">
        <Image class="w-5 h-5" />
      </button>
      <button @click="handleHtmlCopyClick" class="p-2 bg-gray-200 rounded hover:bg-gray-300" title="Копировать HTML">
        <Copy class="w-5 h-5" />
      </button>
      <input
          type="file"
          ref="imageInput"
          @change="handleImageUpload"
          accept="image/*"
          class="hidden"
      />
    </div>
    <textarea
        ref="textArea"
        v-model="content"
        @input="saveToHistory"
        class="w-full h-64 p-2 border rounded"
        placeholder="Начните вводить текст..."
    />
  </div>
</template>
<!--  <header class="header container">-->
<!--    <div class="button-group">-->
<!--      <IconButton @click="handleBackClick">-->
<!--        <BackIcon/>-->
<!--      </IconButton>-->
<!--      <IconButton @click="handleForwardClick">-->
<!--        <ForwardIcon/>-->
<!--      </IconButton>-->
<!--      <IconButton @click="handlerNonStyleClick">-->
<!--        <TextIcon/>-->
<!--      </IconButton>-->
<!--      <IconButton @click="handleCapsClick">-->
<!--        <CapsIcon/>-->
<!--      </IconButton>-->
<!--      <IconButton @click="handleImageUploadClick">-->
<!--        <ImageIcon/>-->
<!--      </IconButton>-->
<!--      <TextButton @click="handleHtmlCopyClick">-->
<!--      </TextButton>-->
<!--    </div>-->



<!--  <main>-->
<!--    <Main ref="mainComponent"/>-->
<!--  </main>-->


<style scoped>
.header {
  padding-top: 77px;
}

.button-group {
  display: flex;
  gap: 12px;
}
</style>
<script setup>
import { ref, nextTick } from 'vue'
import { Undo2, Redo2, Heading2, Type, Image, Copy } from 'lucide-vue-next'

const content = ref('')
const history = ref([])
const historyIndex = ref(-1)
const textArea = ref(null)
const imageInput = ref(null)

const saveToHistory = () => {
  const newHistory = history.value.slice(0, historyIndex.value + 1)
  newHistory.push(content.value)
  history.value = newHistory
  historyIndex.value = newHistory.length - 1
}

const handleBackClick = () => {
  if (historyIndex.value > 0) {
    historyIndex.value--
    content.value = history.value[historyIndex.value]
  }
}

const handleForwardClick = () => {
  if (historyIndex.value < history.value.length - 1) {
    historyIndex.value++
    content.value = history.value[historyIndex.value]
  }
}

const insertHeader = () => {
  const textarea = textArea.value
  const start = textarea.selectionStart
  const end = textarea.selectionEnd
  const selectedText = content.value.substring(start, end)

  content.value =
      content.value.substring(0, start) +
      `## ${selectedText}` +
      content.value.substring(end)

  saveToHistory()

  nextTick(() => {
    textarea.selectionStart = start + 3
    textarea.selectionEnd = end + 3
    textarea.focus()
  })
}

const convertToParagraph = () => {
  const textarea = textArea.value
  const start = textarea.selectionStart
  const end = textarea.selectionEnd
  const selectedText = content.value.substring(start, end)

  content.value =
      content.value.substring(0, start) +
      selectedText.replace(/^#+\s*/, '') +
      content.value.substring(end)

  saveToHistory()

  nextTick(() => {
    textarea.selectionStart = start
    textarea.selectionEnd = end
    textarea.focus()
  })
}

const triggerImageUpload = () => {
  imageInput.value.click()
}

const handleImageUpload = (e) => {
  const file = e.target.files[0]
  if (file) {
    const reader = new FileReader()
    reader.onload = (event) => {
      const textarea = textArea.value
      const start = textarea.selectionStart

      content.value =
          content.value.substring(0, start) +
          `\n![${file.name}](${event.target.result})\n` +
          content.value.substring(start)

      saveToHistory()
    }
    reader.readAsDataURL(file)
  }
}

const handleHtmlCopyClick = () => {
  const markdownToHtml = (markdown) => {
    return markdown
        .replace(/^## (.*$)/gim, '<h2>$1</h2>')
        .replace(/^### (.*$)/gim, '<h3>$1</h3>')
        .replace(/!\[(.*?)\]\((.*?)\)/g, '<img alt="$1" src="$2" />')
        .replace(/\n/g, '<br />')
  }

  const htmlContent = markdownToHtml(content.value)
  navigator.clipboard.writeText(htmlContent)
}
</script>