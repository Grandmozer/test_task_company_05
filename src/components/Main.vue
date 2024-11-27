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
        }
      })
    },
  }
}
</script>