<template>
  <div>
    <div id="editorjs"></div>
  </div>
</template>

<script>
import EditorJS from '@editorjs/editorjs'

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
        placeholder: 'Начните писать здесь...',
        tools: {
          // Вы можете добавить дополнительные плагины здесь
        }
      })
    },
    async getHtmlContent() {
      const content = await this.editor.save()
      // Простой метод конвертации данных EditorJS в HTML
      const htmlContent = content.blocks.map(block => {
        switch(block.type) {
          case 'paragraph':
            return `<p>${block.data.text}</p>`
          case 'header':
            return `<h${block.data.level}>${block.data.text}</h${block.data.level}>`
            // Добавьте другие типы блоков по необходимости
          default:
            return ''
        }
      }).join('')

      return htmlContent
    }
  }
}
</script>