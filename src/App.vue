<script setup>
import Main from './components/Main.vue'

</script>
<template>

  <header>
    <div class="button-group">
      <IconButton @click="handleBackClick">
        <BackIcon />
      </IconButton>
      <IconButton @click="handleForwardClick">
        <ForwardIcon />
      </IconButton>
      <IconButton @click="handleHeaderClick">
        <Caps />
      </IconButton>
      <IconButton @click="handleParagraphClick">
        <Text />
      </IconButton>
      <IconButton @click="handleImageUploadClick">
        <ImageIcon />
      </IconButton>
      <TextButton @click="handleHtmlCopyClick">
        Copy HTML
      </TextButton>
    </div>
  </header>

  <main>
    <Main />
  </main>
</template>

<style scoped>

</style>
<script>
import BackIcon from '@/components/icons/BackIcon.vue'
import ForwardIcon from '@/components/icons/ForwardIcon.vue'
import ImageIcon from '@/components/icons/ImageIcon.vue'

import IconButton from '@/components/buttons/IconButton.vue'
import TextButton from '@/components/buttons/TextButton.vue'

export default {
  components: {
    IconButton,
    TextButton,
    BackIcon,
    ForwardIcon,
    ImageIcon
  },
  methods: {
    handleBackClick() {
      // Реализация функциональности кнопки "Назад"
    },
    handleForwardClick() {
      // Реализация функциональности кнопки "Вперед"
    },
    handleHeaderClick() {
      // Реализация функциональности кнопки "Заголовок"
    },
    handleParagraphClick() {
      // Реализация функциональности кнопки "Абзац"
    },
    handleImageUploadClick() {
      // Реализация функциональности кнопки "Загрузить картинку"
    },
    handleHtmlCopyClick() {
      // Реализация функциональности кнопки "Копировать HTML"
    }
  }
}
</script>