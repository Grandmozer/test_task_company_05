<template>
  <div class="img-btn-upload__block">
    <button
        @click="triggerFileInput"
        :title="title"
        class="img-btn-upload"
    >
      <slot>
        <ImageIcon />
      </slot>
    </button>
    <input
        type="file"
        ref="fileInput"
        @change="handleFileUpload"
        :accept="accept"
        class="img-btn-upload__input"
    />
  </div>
</template>

<style>
.img-btn-upload {
  width: 42px;
  height: 38px;
  flex-shrink: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  border-radius: 4px;
  background: #282828;}

.img-btn-upload__block{
    position: relative;
  }

.img-btn-upload__input{
    position: absolute;
    opacity: 0;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    cursor: pointer;
    z-index: 5;
  }

.img-btn-upload :deep(svg) {
  width: 24px;
  height: 24px;
}
</style>

<script setup>
import { ref } from 'vue'
import ImageIcon from "@/components/icons/ImageIcon.vue";

const props = defineProps({
  title: {
    type: String,
    default: 'Загрузить изображение'
  },
  accept: {
    type: String,
    default: 'image/*'
  }
})

const emit = defineEmits(['upload'])

const fileInput = ref(null)

const triggerFileInput = () => {
  fileInput.value.click()
}

const handleFileUpload = (event) => {
  const file = event.target.files[0]
  if (file) {
    emit('upload', file)
  }
  // Очистка value для возможности загрузки того же файла
  event.target.value = ''
}
</script>