<template>
  <button class="icon-btn" @click="$emit('click')">
    <slot></slot>
  </button>
</template>

<style scoped>
.icon-btn {
  width: 42px;
  height: 38px;
  flex-shrink: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  border-radius: 4px;
  background: #282828;
}

.icon-btn :deep(svg) {
  width: 24px;
  height: 24px;
}
</style>

<script>
export default {
  name: 'IconButton',
  emits: ['click']
}

</script>
