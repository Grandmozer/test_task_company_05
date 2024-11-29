<template>
  <div class="editor-container">
    <header class="header button-group">
      <IconButton
          @upload="handleBackClick"
          title="Назад">
        <BackIcon/>
      </IconButton>
      <IconButton
          @upload="handleForwardClick"
          title="Вперед">
        <ForwardIcon/>
      </IconButton>
      <IconButton
          @upload="insertHeader"
          title="Заголовок">
        <TextIcon/>
      </IconButton>
      <IconButton
          @upload="convertToParagraph"
          title="Абзац">
        <CapsIcon/>
      </IconButton>
      <ImageUploadButton
          @upload="handleImageUpload"
          title="Загрузить изображение"
      />
      <TextButton  @click="handleHtmlCopyClick"></TextButton>

    </header>
    <div id="editorjs"></div>
  </div>
</template>
<script>
import EditorJS from '@editorjs/editorjs'
import Header from '@editorjs/header'
import ImageTool from '@editorjs/image'
import Paragraph from '@editorjs/paragraph'
import IconButton from './components/buttons/IconButton.vue'
import TextButton from './components/buttons/TextButton.vue'
import ImageUploadButton from './components/buttons/ImageUploadButton.vue'
import BackIcon from "@/components/icons/BackIcon.vue";
import ForwardIcon from "@/components/icons/ForwardIcon.vue";
import TextIcon from "@/components/icons/TextIcon.vue";
import CapsIcon from "@/components/icons/CapsIcon.vue";


export default {
  components: {
    CapsIcon,
    TextIcon,
    ForwardIcon,
    BackIcon,
    ImageUploadButton
  },
  data() {
    return {
      editor: null,
      history: [],
      historyIndex: -1
    }
  },
  mounted() {
    this.initEditor()
  },
  methods: {
    initEditor() {
      this.editor = new EditorJS({
        holder: 'editorjs',
        placeholder: 'Начните писать здесь...',
        tools: {
          header: {
            class: Header,
            inlineToolbar: ['link']
          },
          image: {
            class: ImageTool,
            config: {
              uploader: {
                uploadByFile: this.uploadImage
              }
            }
          },
          paragraph: {
            class: Paragraph,
            config: {
              preserveBlank: true,
              placeholder: 'Начните писать здесь...'
            }
          }
        },
        onChange: this.saveToHistory,
        onKeyDown: this.handleKeyDown
      })
    },
    handleKeyDown(event) {
      if (event.key === 'Enter') {
        event.preventDefault()
        return false
      }
    },
    async saveToHistory() {
      try {
        const content = await this.editor.save()

        // Усечение истории до текущего индекса
        const newHistory = this.history.slice(0, this.historyIndex + 1)
        newHistory.push(JSON.stringify(content))

        this.history = newHistory
        this.historyIndex = newHistory.length - 1
      } catch (error) {
        console.error('Saving failed', error)
      }
    },
    async handleBackClick() {
      if (this.historyIndex > 0) {
        this.historyIndex--
        await this.restoreFromHistory()
      }
    },
    async handleForwardClick() {
      if (this.historyIndex < this.history.length - 1) {
        this.historyIndex++
        await this.restoreFromHistory()
      }
    },
    async restoreFromHistory() {
      try {
        const savedContent = JSON.parse(this.history[this.historyIndex])
        await this.editor.render(savedContent)
      } catch (error) {
        console.error('Restoring from history failed', error)
      }
    },
    async insertHeader() {
      try {
        const currentBlockIndex = this.editor.blocks.getCurrentBlockIndex()
        if (currentBlockIndex === -1) return

        const savedData = await this.editor.save()
        const currentBlock = savedData.blocks[currentBlockIndex]

        if (!currentBlock) return

        // Удаляем текущий блок и вставляем новый заголовок
        await this.editor.blocks.delete(currentBlockIndex)
        await this.editor.blocks.insert('header', {
          text: currentBlock.data.text,
          level: 2  // или любой другой уровень
        }, {}, currentBlockIndex)

      } catch (error) {
        console.error('Ошибка при создании заголовка:', error)
      }
    },
    async convertToParagraph() {
      try {
        const currentBlockIndex = this.editor.blocks.getCurrentBlockIndex()
        if (currentBlockIndex === -1) return

        const savedData = await this.editor.save()
        const currentBlock = savedData.blocks[currentBlockIndex]

        if (!currentBlock) return

        const selection = window.getSelection()
        const selectedText = selection.toString().trim()

        let textToConvert = selectedText || currentBlock.data.text

        // Очищаем текст от разметки заголовков
        textToConvert = textToConvert.replace(/^#+\s*/, '')

        if (!textToConvert) return

        if (selectedText && selectedText !== currentBlock.data.text) {
          const range = selection.getRangeAt(0)
          const selectedContent = range.extractContents()

          await this.editor.blocks.delete(currentBlockIndex)
          await this.editor.blocks.insert('paragraph', { text: textToConvert }, {}, currentBlockIndex)
        } else {
          await this.editor.blocks.delete(currentBlockIndex)
          await this.editor.blocks.insert('paragraph', { text: textToConvert }, {}, currentBlockIndex)
        }
      } catch (error) {
        console.error('Ошибка при преобразовании в параграф:', error)
      }
    },
    async handleImageUpload(file) {
      try {
        const reader = new FileReader()
        reader.onload = async (event) => {
          await this.editor.blocks.insert('image', {
            file: {
              url: event.target.result
            },
            caption: file.name
          })
        }
        reader.readAsDataURL(file)
      } catch (error) {
        console.error('Image upload failed', error)
      }
    },
    async uploadImage(file) {
      // Локальная загрузка изображения для демонстрации
      return new Promise((resolve) => {
        const reader = new FileReader()
        reader.onload = (event) => {
          resolve({
            success: 1,
            file: {
              url: event.target.result
            }
          })
        }
        reader.readAsDataURL(file)
      })
    },
    async handleHtmlCopyClick() {
      try {
        const content = await this.editor.save()
        const htmlContent = this.convertToHtml(content)
        await navigator.clipboard.writeText(htmlContent)
      } catch (error) {
        console.error('HTML copy failed', error)
      }
    },
    convertToHtml(data) {
      return data.blocks.map(block => {
        switch (block.type) {
          case 'header':
            return `<h${block.data.level}>${block.data.text}</h${block.data.level}>`
          case 'paragraph':
            return `<p>${block.data.text}</p>`
          case 'image':
            return `<img src="${block.data.file.url}" alt="${block.data.caption || ''}" />`
          default:
            return ''
        }
      }).join('\n')
    }
  }
}
</script>

<style scoped>
.editor-container {
  max-width: 800px;
  margin: 0 auto;
}

.button-group {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}
</style>