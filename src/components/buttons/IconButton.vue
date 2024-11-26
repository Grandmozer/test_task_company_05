<script setup>
defineProps({
  icon: {
    type: Object,
    required: true
  }
})
defineEmits(['click'])
</script>


<template>
  <button class="icon-btn" @click="$emit('click')">
    <component :is="icon"/>
  </button>
</template>

<style scoped>
.icon-btn {
  width: 42px;
  height: 38px;
  flex-shrink: 0;
  border-radius: 4px;
  background: #282828;
}

.icon-btn :deep(svg) {
  width: 24px;
  height: 24px;
}
</style>