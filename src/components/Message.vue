<script setup>
  import { ref } from 'vue';
const text = ref("");
const show = (newText) => {
  text.value = newText;
};
const hide = () => {
  text.value = "";
};
defineExpose({ show, hide });
</script>

<template>
  <article class="message" :class="{ 'message--hidden': !text }">
    <div class="message-header">
      <p>Error</p>
      <button class="delete" @click="hide"></button>
    </div>

    <div class="message-body">{{ text }}</div>
    <slot :text="text"></slot>
  </article>
</template>

<style scoped lang="scss">
  .message {
    transform-origin: top center;
    transition-property: opacity, transform;
    transition-duration: 0.3s;

    &--hidden {
      transform: scaleY(0);
      opacity: 0;
      pointer-events: none;
    }
  }
</style>
