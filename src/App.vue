<template>
  <div class="editor-container">
    <header class="header button-group">
      <button @click="handleBackClick" title="Назад">Назад</button>
      <button @click="handleForwardClick" title="Вперед">Вперед</button>
      <button @click="insertHeader" title="Заголовок">Заголовок</button>
      <button @click="convertToParagraph" title="Абзац">Абзац</button>
      <ImageUploadButton
          @upload="handleImageUpload"
          title="Загрузить изображение"
      />
      <button @click="handleHtmlCopyClick">Копировать HTML</button>
    </header>
    <div id="editorjs"></div>
  </div>
</template>
<script>
import EditorJS from '@editorjs/editorjs'
import Header from '@editorjs/header'
import ImageTool from '@editorjs/image'
import ImageUploadButton from './components/buttons/ImageUploadButton.vue'

export default {
  components: {
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
          }
        },
        onChange: this.saveToHistory
      })
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
        const selection = await this.editor.save()
        const blocks = selection.blocks

        if (blocks.length > 0) {
          const currentBlock = blocks[blocks.length - 1]

          // Преобразуем последний блок в заголовок
          await this.editor.blocks.update(currentBlock.id, {
            type: 'header',
            data: {
              text: currentBlock.data.text,
              level: 2
            }
          })
        }
      } catch (error) {
        console.error('Insert header failed', error)
      }
    },
    async convertToParagraph() {
      try {
        // Получаем текущий блок
        const currentBlockIndex = this.editor.blocks.getCurrentBlockIndex()
        if (currentBlockIndex === -1) return

        const currentBlock = await this.editor.blocks.getBlockByIndex(currentBlockIndex)

        // Проверяем, если блок является заголовком
        if (currentBlock.name === 'header') {
          // Преобразуем заголовок в параграф
          await this.editor.blocks.update(currentBlock.id, {
            type: 'paragraph',
            data: {
              text: currentBlock.data.text
            }
          })
        }
      } catch (error) {
        console.error('Convert to paragraph failed', error)
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