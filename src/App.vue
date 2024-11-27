<script setup>
import Main from './components/Main.vue'

</script>
<template>

  <header class="header container">
    <div class="button-group">
      <IconButton @click="handleBackClick">
        <BackIcon/>
      </IconButton>
      <IconButton @click="handleForwardClick">
        <ForwardIcon/>
      </IconButton>
      <IconButton @click="handlerNonStyleClick">
        <TextIcon />
      </IconButton>
      <IconButton @click="handleCapsClick">
        <CapsIcon />
      </IconButton>
      <IconButton @click="handleImageUploadClick">
        <ImageIcon />
      </IconButton>
      <TextButton @click="handleHtmlCopyClick">
      </TextButton>
    </div>
  </header>

  <main>
    <Main ref="mainComponent" />
  </main>
</template>

<style scoped>
.header{
  padding-top: 77px ;
}
.button-group{
  display: flex;
  gap: 12px;
}
</style>
<script>
import EditorJS from '@editorjs/editorjs'

import BackIcon from '@/components/icons/BackIcon.vue'
import ForwardIcon from '@/components/icons/ForwardIcon.vue'
import ImageIcon from '@/components/icons/ImageIcon.vue'
import TextIcon from "@/components/icons/TextIcon.vue";
import CapsIcon from "@/components/icons/CapsIcon.vue";

import IconButton from '@/components/buttons/IconButton.vue'
import TextButton from '@/components/buttons/TextButton.vue'

export default {
  components: {
    IconButton,
    TextButton,
    BackIcon,
    ForwardIcon,
    ImageIcon,
    CapsIcon,
    TextIcon,
  },
  data() {
    return {
      editor: null,
      history: [],
      currentIndex: -1,
      lastRemovedLine: null
    }
  },
  mounted() {
    this.initEditor()
  },
  methods: {
    initEditor() {
      this.editor = new EditorJS({
        holder: 'editorjs',
        data: {
          blocks: []
        },
        onChange: () => {
          this.saveCurrentState()
        }
      })
    },
    saveCurrentState() {
      this.editor.save().then((savedData) => {
        this.saveToHistory(savedData)
      }).catch((error) => {
        console.error('Saving error', error)
      })
    },
    saveToHistory(state) {
      if (this.currentIndex < this.history.length - 1) {
        this.history = this.history.slice(0, this.currentIndex + 1)
      }

      this.history.push(state)
      this.currentIndex++
    },

    undo() {
      if (this.currentIndex > 0) {
        const currentState = this.history[this.currentIndex]
        this.lastRemovedLine = currentState.blocks[currentState.blocks.length - 1]

        currentState.blocks.pop()

        this.currentIndex--
        return currentState
      }
    },

    redo() {
      if (this.currentIndex < this.history.length - 1 && this.lastRemovedLine) {
        this.currentIndex++
        const currentState = this.history[this.currentIndex]

        currentState.blocks.push(this.lastRemovedLine)
        this.lastRemovedLine = null

        return currentState
      }
    },
    handleBackClick() {
      const previousState = this.undo()
      if (previousState) {
        this.restoreState(previousState)
      }
    },
    handleForwardClick() {
      const nextState = this.redo()
      if (nextState) {
        this.restoreState(nextState)
      }
    },
    handleCapsClick() {
      // Реализация функциональности кнопки "Заголовок"
    },
    handlerNonStyleClick() {
      // Реализация функциональности кнопки "Абзац"
    },
    handleImageUploadClick() {
      // Реализация функциональности кнопки "Загрузить картинку"
    },
    handleHtmlCopyClick() {
      // Получаем ссылку на компонент Main через refs
      const mainComponent = this.$refs.mainComponent;

      // Проверяем, что компонент существует
      if (!mainComponent) {
        console.error('Main component not found');
        return;
      }

      // Получаем корневой элемент компонента Main
      const mainElement = mainComponent.$el;

      // Создаем временный текстовый элемент для копирования
      const tempTextArea = document.createElement('textarea');
      tempTextArea.value = mainElement.innerHTML.trim();

      // Добавляем элемент в документ
      document.body.appendChild(tempTextArea);

      // Выделяем текст
      tempTextArea.select();

      try {
        // Копируем выделенный текст
        const successful = document.execCommand('copy');
        const msg = successful ? 'Успешно скопировано' : 'Не удалось скопировать';

        // Можно добавить всплывающее уведомление
        alert(msg);
      } catch (err) {
        console.error('Ошибка копирования', err);
        alert('Не удалось скопировать HTML');
      }

      // Удаляем временный элемент
      document.body.removeChild(tempTextArea);
    },  restoreState(state) {
      // Очищаем текущее содержимое редактора
      this.editor.clear()

      // Загружаем сохраненное состояние
      this.editor.render(state)
    }
  },
  computed: {
    canUndo() {
      return this.currentIndex > 0
    },
    canRedo() {
      return this.currentIndex < this.history.length - 1 && this.lastRemovedLine !== null
    }
}
}

</script>