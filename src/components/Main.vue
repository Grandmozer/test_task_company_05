<template>
  <div>
    <div id="editorjs"></div>
  </div>
</template>

<script>
import EditorJS from '@editorjs/editorjs'
import Header from '@editorjs/header'
import ImageTool from '@editorjs/image'

export default {
  data() {
    return {
      editor: null
    }
  },
  mounted() {
    this.initEditor()
  },
  methods: {
    initEditor() {
      this.editor = new EditorJS({
        holder: 'editorjs',
        tools: {
          header: {
            class: Header,
            inlineToolbar: ['link']
          },
          image: {
            class: ImageTool,
            config: {
              // Здесь вы можете настроить загрузку изображений
              // Например, через API или локальную загрузку
              endpoints: {
                byFile: '/upload', // URL для загрузки файлов
                byUrl: '/fetch-url' // URL для загрузки по ссылке
              }
            }
          }
        }
      })
    },
    async saveContent() {
      try {
        const content = await this.editor.save()
        console.log('Saved content:', content)
        // Здесь вы можете отправить контент на сервер или обработать его
      } catch (error) {
        console.error('Saving failed:', error)
      }
    },
    clearContent() {
      this.editor.clear()
    }
  },
  beforeUnmount() {
    // Уничтожаем редактор при закрытии компонента
    if (this.editor) {
      this.editor.destroy()
    }
  }
}
</script>

<style scoped>
</style>